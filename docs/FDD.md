# FDD — Sistema de webhooks de pedidos

Fontes dos contratos e pontos de integração: [Tracker de rastreabilidade](TRACKER.md).

## Contexto e motivação técnica

`OrderService.changeStatus` usa uma transação Prisma para validar a transição, ajustar estoque, atualizar `orders` e criar `order_status_history` (`src/modules/orders/order.service.ts`). O envio HTTP não pode ocorrer nela; a outbox deve ser gravada no mesmo commit.

## Objetivos técnicos

- Preservar atomicidade entre status e evento.
- Fazer primeira tentativa em menos de 10 s sob operação saudável, com polling de 2 s.
- Entregar at-least-once com assinatura verificável, retries e DLQ.

## Escopo e exclusões

Inclui módulo de configuração, outbox, worker, histórico, rotação e replay. Email, dashboard, rate limiting, arquivamento e múltiplos workers ficam fora desta fase.

## Modelo de dados proposto

Esta é uma **proposta de implementação** das entidades mencionadas, sujeita à revisão do RFC; nomes de colunas além dos explicitamente ditos não foram decididos na call.

| Entidade | Campos mínimos e função |
|---|---|
| `webhook_endpoints` | UUID, `customer_id`, URL HTTPS, secret atual/anterior e expiração da anterior, ativo, status inscritos |
| `webhook_outbox` | UUID/event_id, endpoint, order_id, snapshot JSON, estado, criação, próxima tentativa, contador e último erro; índice por estado/criação |
| `webhook_deliveries` | Evento, tentativa, horário, sucesso/falha, payload, resposta e duração para consulta histórica |
| `webhook_dead_letter` | UUID, evento original, payload, motivo, horário e referência ao endpoint |

A transcrição usa estados “pendente, processando, falhou, entregue” e tabela separada para falha definitiva. O mapeamento exato de enum e colunas fica a cargo da migration, sem criar outro comportamento de negócio.

## Fluxos detalhados

### 1. Criação do evento na outbox

1. `changeStatus` valida a transição via `canTransition`; executa ajuste de estoque, update e histórico como já faz. `publishWebhookEvent(tx, order, fromStatus, toStatus)` é chamado **antes do commit**.
2. A função seleciona endpoints ativos do `customer_id` que incluem `toStatus`. Se não houver inscrição, não insere linha.
3. Para cada endpoint, gera UUID e snapshot do estado da transição, sem items, e insere na outbox usando o mesmo `tx`. O snapshot deve refletir o momento da mudança, mesmo se o pedido mudar novamente.
4. Falha de inserção rejeita a transação inteira. A criação inicial do pedido não está explicitamente incluída: o gatilho decidido foi mudança de status em `changeStatus`, não `create`.

### 2. Worker e entrega

Processo planejado `src/worker.ts` cria PrismaClient próprio com a mesma `DATABASE_URL` e chama o processador do módulo. A cada 2 s lê lote pequeno de pendentes antigos por `created_at`, envia um a um e registra resultado/duração no histórico. **Detalhe necessário para a ordem por pedido:** não iniciar um evento posterior do mesmo `order_id` enquanto o anterior estiver pendente ou aguardando retry. Um worker isolado não garante isso sozinho, pois o evento posterior poderia ser entregue durante o backoff do anterior. Ao mover o anterior para a DLQ, a sequência fica interrompida e o cliente deve reconciliar o estado. Esta regra de bloqueio é uma proposta técnica para cumprir a ordenação por pedido; não há garantia global.

**Claim e recuperação propostos:** marcar atomicamente a linha selecionada como `processing` antes do HTTP e ignorar um claim que já tenha mudado de estado. Após reinício, reabrir claims sem progresso para nova tentativa; isso pode duplicar o envio, portanto o `event_id` permanece o mesmo. O prazo de abandono do claim e o mecanismo SQL exato precisam ser definidos na implementação e testados com queda do processo. Esta é uma concretização da outbox e da garantia at-least-once, não um parâmetro fechado na reunião.

Serializar o snapshot uma vez para bytes UTF-8; recusar corpo acima de 64 KB; calcular HMAC-SHA256 sobre **esses bytes** e usá-los sem alteração no HTTP. Headers: `Content-Type: application/json`, `X-Event-Id`, `X-Signature`, `X-Timestamp` (horário do envio) e `X-Webhook-Id`. Timeout: 10 s. O formato textual da assinatura, tratamento de códigos HTTP e retenção da resposta não foram detalhados na reunião; o contrato abaixo é uma convenção proposta para revisão.

### 3. Retry e DLQ

Após erro de rede, timeout ou resposta não aceita, agendar nova tentativa nos intervalos de 1 min, 5 min, 30 min, 2 h e 12 h; quando esgotada a política, copiar payload, motivo e timestamp para `webhook_dead_letter`. **Bloqueio para implementação:** a reunião disse “5 tentativas” e enumerou cinco intervalos. Definir se são cinco envios totais ou cinco retries após o envio inicial antes de fixar o contador/cronograma. A proposta operacional é contar o envio inicial como tentativa 1 e aplicar apenas os intervalos que precedem as tentativas restantes; isto exige validação dos revisores.

### 4. Replay

`POST /admin/webhooks/dead-letter/:id/replay` passa por `authenticate` e `requireRole('ADMIN')`; registra o usuário que realizou o replay e reinsere pendência para processamento. Preservar o event_id original no replay é **proposta** para manter a deduplicação; a call não especificou essa semântica. Fechar na revisão técnica.

## Contratos públicos

Os caminhos de CRUD abaixo são **proposta de concretização** das operações POST/PATCH/DELETE/GET discutidas na reunião. Todos usam `Authorization: Bearer <jwt>`. O JWT atual não contém `customer_id`; por isso o path o declara explicitamente. **Antes de expor os endpoints**, definir e implementar autorização de acesso ao cliente. Replay exige `ADMIN`. A API existente retorna JSON diretamente e erros no envelope `{ "error": { "code", "message" } }` (`src/modules/orders/order.controller.ts`, `src/middlewares/error.middleware.ts`).

### POST /customers/:customerId/webhooks

```http
POST /customers/11111111-1111-4111-8111-111111111111/webhooks
Authorization: Bearer <jwt>
Content-Type: application/json

{"url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"]}
```

`201 Created` (secret exibida somente nesta resposta, proposta de segurança para revisão):
```json
{"id":"22222222-2222-4222-8222-222222222222","customerId":"11111111-1111-4111-8111-111111111111","url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"],"active":true,"secret":"<generated-secret>"}
```
Erros: 400 URL/status inválido, 401 sem JWT, 403 sem acesso ao cliente, 404 cliente inexistente. HTTPS é obrigatório.

### GET /customers/:customerId/webhooks

```http
GET /customers/11111111-1111-4111-8111-111111111111/webhooks
Authorization: Bearer <jwt>
```

`200 OK`:
```json
{"data":[{"id":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"],"active":true}]}
```
Secret omitida. Erros: 401, 403, 404.

### PATCH /customers/:customerId/webhooks/:id

```http
PATCH /customers/11111111-1111-4111-8111-111111111111/webhooks/22222222-2222-4222-8222-222222222222
Authorization: Bearer <jwt>
Content-Type: application/json

{"statuses":["DELIVERED"],"active":false}
```

`200 OK`:
```json
{"id":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks/orders","statuses":["DELIVERED"],"active":false}
```
Erros: 400, 401, 403, 404. `DELETE` no mesmo path retorna 204 sem corpo (convenção proposta).

### POST /customers/:customerId/webhooks/:id/rotate-secret

```http
POST /customers/11111111-1111-4111-8111-111111111111/webhooks/22222222-2222-4222-8222-222222222222/rotate-secret
Authorization: Bearer <jwt>
```

`200 OK`:
```json
{"id":"22222222-2222-4222-8222-222222222222","secret":"<new-secret>","previousSecretValidUntil":"2026-09-24T12:00:00.000Z"}
```
Erros: 401, 403, 404. A reunião definiu validade paralela por 24 h, mas não especificou se o produtor assina com uma ou ambas as secrets, nem como o consumidor escolhe a versão. O exemplo apenas expõe a expiração; a semântica de assinatura durante a janela deve ser fechada na revisão.

### GET /webhooks/:id/deliveries

```http
GET /webhooks/22222222-2222-4222-8222-222222222222/deliveries
Authorization: Bearer <jwt>
```

`200 OK`:
```json
{"data":[{"eventId":"33333333-3333-4333-8333-333333333333","success":false,"payload":{"event_id":"33333333-3333-4333-8333-333333333333"},"response":{"status":503},"durationMs":10000}]}
```
Proposta: limitar a 100 entradas recentes; a fala pediu “últimos 100” como exemplo, não fixou paginação. Erros: 401, 403, 404.

### POST /admin/webhooks/dead-letter/:id/replay

```http
POST /admin/webhooks/dead-letter/44444444-4444-4444-8444-444444444444/replay
Authorization: Bearer <admin-jwt>
```

`202 Accepted` (proposta):
```json
{"deadLetterId":"44444444-4444-4444-8444-444444444444","status":"pending"}
```
Erros: 401, 403 para não ADMIN, 404. Registrar `req.user.id` na auditoria.

### POST outbound enviado ao cliente

```http
POST /webhooks/orders
Content-Type: application/json
X-Event-Id: 33333333-3333-4333-8333-333333333333
X-Webhook-Id: 22222222-2222-4222-8222-222222222222
X-Timestamp: 2026-09-23T12:00:00.000Z
X-Signature: <hmac-sha256>

{"event_id":"33333333-3333-4333-8333-333333333333","event_type":"order.status_changed","timestamp":"2026-09-23T12:00:00.000Z","order_id":"55555555-5555-4555-8555-555555555555","order_number":"ORD-000123","from_status":"PROCESSING","to_status":"SHIPPED","customer_id":"11111111-1111-4111-8111-111111111111","total_cents":2590}
```

O cliente retorna resposta HTTP; **convenção proposta**: qualquer 2xx confirma entrega, demais respostas/timeout acionam retry. Não enviar `items`. O cliente pode consultar `GET /orders/:id` para detalhes.

## Matriz de erros previstos

| Código | HTTP/efeito |
|---|---|
| `WEBHOOK_INVALID_URL` | 400; URL sem HTTPS |
| `WEBHOOK_NOT_FOUND` | 404; cadastro não localizado |
| `WEBHOOK_INVALID_STATUS_FILTER` | 400; status fora do enum |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | falha de entrega/DLQ conforme política; >64 KB |
| `WEBHOOK_DELIVERY_TIMEOUT` | falha recuperável após 10 s |
| `WEBHOOK_DELIVERY_FAILED` | falha recuperável por resposta não aceita |
| `WEBHOOK_DEAD_LETTER_NOT_FOUND` | 404 no replay |
| `WEBHOOK_CUSTOMER_ACCESS_DENIED` | 403; depende da regra de vínculo pendente |

Os códigos específicos são propostas sob o prefixo `WEBHOOK_` definido para o módulo. Falhas comuns de JWT continuam `UNAUTHORIZED`/`FORBIDDEN` do middleware existente.

## Resiliência

Timeout 10 s; backoff finito conforme fluxo de retry; DLQ separada e replay manual. A falha de um destino não reverte o status do pedido. Não há email fallback nem rate limiting nesta fase. A fila precisa preservar o mesmo `event_id` em reenvios para permitir deduplicação.

## Observabilidade

Usar Pino existente para logs estruturados de `event_id`, `webhook_id`, tentativa, duração, resultado e replay/admin; nunca registrar secrets. Medir idade da pendência, número pendente/DLQ, latência commit→primeira tentativa, duração HTTP e taxa de entregas/falhas. Propagar `requestId` se disponível e correlacionar API→outbox→worker pelo `event_id` em tracing. Logs/tracing e nomes exatos das métricas são **instrumentação proposta** para verificar o alvo de latência e diagnosticar falhas, não decisões textuais da reunião. O Pino e o request ID existentes estão em `src/shared/logger/index.ts` e `src/middlewares/error.middleware.ts`.

## Integração com o sistema existente

| Arquivo real | Integração |
|---|---|
| `src/modules/orders/order.service.ts` | Chamar `publishWebhookEvent(tx, refreshed, from, to)` dentro de `changeStatus`, após update/histórico e antes de retornar/commitar. |
| `src/modules/orders/order.status.ts` | Reusar transições válidas; usar `toStatus` como filtro. |
| `prisma/schema.prisma` | Adicionar modelos de endpoint, outbox, delivery e DLQ, relações e índices; preservar enum `OrderStatus` e UUID do projeto. |
| `src/middlewares/auth.middleware.ts` | Reusar `authenticate` e `requireRole('ADMIN')` no replay; resolver lacuna de autorização por cliente. |
| `src/shared/errors/app-error.ts` e `src/middlewares/error.middleware.ts` | Criar erros `WEBHOOK_*` sobre `AppError`, usando envelope central. |
| `src/shared/logger/index.ts` | Reusar Pino e ampliar redaction para secrets do novo módulo antes de registrar objetos. |
| `src/config/database.ts`, `src/server.ts`, `src/routes/index.ts` | Worker com PrismaClient próprio; rotas do novo módulo montadas na API. |

## Dependências e compatibilidade

Node/TypeScript, Express, Prisma/MySQL, Zod, Pino e JWT já compõem o projeto (`package.json`, `prisma/schema.prisma`). A feature adiciona migration e processo operacional; não altera contrato de resposta de `PATCH /orders/:id/status`, apenas seu efeito transacional. Clientes devem deduplicar por `X-Event-Id`.

## Critérios de aceite técnicos

1. Teste transacional prova rollback conjunto de status/histórico/estoque/outbox e ausência de evento sem inscrição.
2. Com destino saudável, primeira tentativa ocorre em menos de 10 s; com destino lento, timeout de 10 s não bloqueia `changeStatus`.
3. Assinatura HMAC dos bytes enviados verifica com a secret do endpoint, inclusive rotação de 24 h; HTTP e >64 KB são rejeitados.
4. Falhas seguem a contagem aprovada, acabam na DLQ e replay `ADMIN` é auditado.
5. Entrega duplicada mantém `X-Event-Id` e ordenação por pedido é validada com um worker.

## Riscos e mitigação

Aumento de latência da transação pela escrita extra: manter inserção enxuta e índices adequados. Lentidão/acúmulo do worker: lotes pequenos e observabilidade. Vazamento de secret em logs: redaction e revisão de Sofia antes do deploy. Sem associação usuário→cliente no JWT, o CRUD não deve ser liberado até definir a autorização na revisão; ver `src/middlewares/auth.middleware.ts`.
