# RFC — Webhooks de mudanças de status de pedidos

| Campo | Valor |
|---|---|
| Autor | Equipe de engenharia; documento elaborado com IA a partir da reunião |
| Status | Proposta para revisão |
| Data | 2026-09-23 (elaboração; data da reunião não informada) |
| Revisores | Larissa, Marcos, Bruno, Diego e Sofia |

## TL;DR

Registrar um snapshot de evento na mesma transação da mudança de status e entregá-lo via worker separado em polling de 2 s. Assinar cada envio com HMAC-SHA256 por endpoint, retentar falhas com backoff e mover falhas definitivas para DLQ. O consumidor deduplica por `X-Event-Id`. [09:06] Diego; [09:17–09:26] Diego, Sofia e Larissa.

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo consultam `GET /orders` repetidamente e pedem notificação em menos de 10 s. O fluxo é só outbound. [09:00–09:03] Marcos e Sofia. `changeStatus` já atualiza pedido, histórico e estoque na transação (`src/modules/orders/order.service.ts`); HTTP nesse caminho acoplaria a operação ao destino. [09:04] Bruno.

## Proposta técnica

1. Persistir configuração por `customer_id`, URL HTTPS, secret individual, estado ativo e filtro de status. Para cada transição, filtrar inscrições e inserir um snapshot por endpoint na transação do pedido. [09:21] Sofia; [09:31–09:34] Marcos e Bruno; [09:40–09:41] Bruno; [09:52] Larissa.
2. Um único worker separado consulta pendências antigas a cada 2 s em lotes pequenos. Envia JSON com `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id`; timeout 10 s e limite 64 KB. [09:08–09:13] Diego e Larissa; [09:23–09:25] Sofia e Diego; [09:42–09:45] Diego e Sofia.
3. Retry finito e dead letter persistida, com replay administrativo. Histórico de entregas disponível ao cliente. [09:17–09:19] Diego e Larissa; [09:34–09:36] Marcos e Sofia.
4. Reusar Express, Prisma, Zod, Pino, `AppError` e o padrão de módulos. [09:27–09:30] Bruno e Larissa.

Contratos e integração ficam no [FDD](FDD.md).

## Alternativas consideradas

| Alternativa | Trade-off do descarte |
|---|---|
| HTTP síncrono no `OrderService` | Destino lento ou indisponível bloquearia a transação. [09:03–09:06] Bruno e Diego |
| Redis Streams | Acrescentaria infraestrutura para um time pequeno. [09:07] Larissa e Diego |
| Trigger SQL para despertar worker | Não notifica diretamente processo externo no MySQL. [09:09] Bruno e Diego |
| Exactly-once distribuído | Exigiria coordenação entre produtor e consumidor. [09:25] Diego |

## Questões em aberto

1. **Contagem de retry:** “5 tentativas” e cinco intervalos de backoff aparecem juntos. Confirmar se o envio inicial está incluído. [09:15–09:17] Diego e Larissa.
2. **Autorização por cliente:** o `customer_id` virá do body ou path; o JWT atual representa usuário, não cliente. A reunião não definiu como restringir acesso entre clientes. [09:31–09:33] Marcos, Bruno e Larissa; `src/middlewares/auth.middleware.ts`.
3. **Rate limiting de saída:** observar e decidir em outra fase. [09:38–09:39] Diego e Larissa.
4. **Escala/ordenação:** particionamento por `order_id` ou lock apenas se houver múltiplos workers. [09:12–09:13] Diego e Larissa.

## Impacto e riscos

Novas tabelas de configuração, outbox, tentativas e DLQ; escrita extra em `changeStatus`; processo separado. Falha da inserção na outbox deve abortar a transição. [09:40–09:41] Bruno. Acúmulo de pendências pode elevar latência; índice por estado/criação, lotes pequenos e métricas reduzem o risco. [09:07–09:08] Bruno e Diego. Estimativa: três sprints, com dois dias úteis de revisão de segurança antes do deploy. [09:45–09:47] Larissa e Sofia.

## Decisões relacionadas

[ADR-001](adrs/ADR-001-outbox-no-mysql.md) · [ADR-002](adrs/ADR-002-worker-separado-em-polling.md) · [ADR-003](adrs/ADR-003-retry-e-dead-letter.md) · [ADR-004](adrs/ADR-004-hmac-e-rotacao-de-secret.md) · [ADR-005](adrs/ADR-005-at-least-once-e-event-id.md) · [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
