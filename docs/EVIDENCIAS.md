# Inventário de evidências — desafio 05

Este catálogo foi levantado de `TRANSCRICAO.md` integral e do código do fork. **T** identifica fala; **C** identifica fato do código. IDs são estáveis para revisão documental; decisões aprovadas, alternativas e sugestões não são confundidas.

## Transcrição

| ID | Classe | Evidência | Localização | Destino |
|---|---|---|---|---|
| T-01 | Problema | Atlas, MaxDistribuição e Nova Cargo pedem aviso de status; hoje fazem polling | [09:00] Marcos | PRD, RFC |
| T-02 | Meta | Menos de 10 s atende “tempo real” | [09:02] Marcos | PRD, RFC, FDD |
| T-03 | Restrição | Somente webhook outbound | [09:02] Marcos | PRD |
| T-04 | Alternativa descartada | HTTP síncrono bloquearia transação/rollback | [09:04] Bruno | ADR-001, RFC |
| T-05 | Decisão | Outbox na transação SQL de pedido e histórico | [09:06] Diego | ADR-001, FDD |
| T-06 | Alternativa descartada | Redis adicionaria infraestrutura | [09:07] Diego | ADR-001, RFC |
| T-07 | Requisito técnico | Índice em estado/criação; batch pequeno | [09:08] Diego | FDD, RFC |
| T-08 | Exclusão | Arquivamento após ~30 dias fora desta feature | [09:08] Diego | PRD |
| T-09 | Decisão | Polling a cada 2 s | [09:09] Diego | ADR-002, FDD |
| T-10 | Alternativa descartada | Trigger MySQL não notifica processo externo | [09:09] Diego | ADR-002, RFC |
| T-11 | Decisão | Worker em processo separado | [09:11] Diego | ADR-002, FDD |
| T-12 | Limite | Um worker e ordem por pedido; sem ordem global | [09:12] Diego | ADR-002, PRD, FDD |
| T-13 | Futuro | Particionamento/lock por order_id se escalar | [09:13] Diego | RFC |
| T-14 | Decisão | Backoff finito; retry ilimitado descartado | [09:15] Diego | ADR-003, RFC |
| T-15 | Alternativa descartada | Três tentativas insuficientes | [09:16] Diego | ADR-003 |
| T-16 | Decisão ambígua | “5 tentativas” e intervalos 1m/5m/30m/2h/12h | [09:17] Larissa | ADR-003, RFC, FDD |
| T-17 | Decisão | DLQ em tabela separada com payload/motivo | [09:18] Diego | ADR-003, FDD |
| T-18 | Decisão | Replay manual via endpoint admin | [09:19] Larissa | ADR-003, FDD |
| T-19 | Decisão | HMAC-SHA256 sobre corpo do request | [09:20] Sofia | ADR-004, FDD |
| T-20 | Decisão | Secret individual por endpoint e rotação com 24 h | [09:21] Sofia | ADR-004, PRD, FDD |
| T-21 | Restrição | HTTPS obrigatório | [09:23] Sofia | PRD, FDD |
| T-22 | Restrição | Payload máximo 64 KB; erro, sem truncar | [09:24] Larissa | PRD, FDD |
| T-23 | Decisão | At-least-once e UUID em X-Event-Id | [09:25] Diego | ADR-005, FDD |
| T-24 | Alternativa descartada | Exactly-once exigiria coordenação | [09:25] Diego | ADR-005, RFC |
| T-25 | Decisão | Módulo controller/service/repository/routes/schemas | [09:27] Bruno | ADR-006, FDD |
| T-26 | Decisão | Prefixo WEBHOOK_ para erros | [09:29] Larissa | ADR-006, FDD |
| T-27 | Decisão | Reusar Pino, AppError, middleware, Zod | [09:30] Larissa | ADR-006, FDD |
| T-28 | Requisito | POST cadastro com secret gerada e filtro | [09:31] Marcos | PRD, FDD |
| T-29 | Correção | JWT representa usuário; customer_id vem de body/path | [09:32] Larissa | RFC, FDD |
| T-30 | Requisito | GET/PATCH/DELETE de configuração | [09:33] Bruno | PRD, FDD |
| T-31 | Requisito | Filtrar status antes de inserir na outbox | [09:34] Bruno | PRD, FDD |
| T-32 | Requisito | Histórico dos envios: resultado, payload, response, duração | [09:34] Marcos | PRD, FDD |
| T-33 | Decisão | Replay requer ADMIN e auditoria do autor | [09:36] Sofia | PRD, FDD |
| T-34 | Exclusão | Email ao cliente fica para outra fase | [09:37] Larissa | PRD |
| T-35 | Questão aberta | Rate limiting de saída a observar | [09:39] Larissa | RFC, PRD |
| T-36 | Exclusão | Dashboard visual fora; somente API | [09:40] Larissa | PRD |
| T-37 | Integração | publishWebhookEvent recebe tx dentro de changeStatus | [09:41] Bruno | ADR-001, FDD |
| T-38 | Restrição | Timeout de envio: 10 s | [09:42] Diego | PRD, FDD |
| T-39 | Contrato | JSON com event_id, tipo, timestamp, IDs, status e total; sem items | [09:43] Diego | FDD |
| T-40 | Contrato | X-Event-Id, X-Signature, X-Timestamp, Content-Type | [09:44] Diego | FDD |
| T-41 | Contrato | X-Webhook-Id | [09:44] Sofia | FDD |
| T-42 | Planejamento | Três sprints; dois dias de revisão de segurança | [09:46] Larissa | PRD, RFC |
| T-43 | Planejamento | Sofia requer dois dias úteis antes de deploy | [09:46] Sofia | PRD |
| T-44 | Decisão | UUID na outbox, como no projeto | [09:51] Larissa | FDD |
| T-45 | Decisão | Snapshot do payload na inserção | [09:52] Larissa | ADR-001, FDD |

## Código

| ID | Arquivo e símbolo | Fato observado | Destino |
|---|---|---|---|
| C-01 | `src/modules/orders/order.service.ts` — `changeStatus` | Transação Prisma envolve status, histórico e ajuste de estoque | ADR-001, RFC, FDD |
| C-02 | `src/modules/orders/order.status.ts` — `canTransition` | Máquina de estados define transições válidas | FDD |
| C-03 | `prisma/schema.prisma` — `OrderStatus`, `Order` | MySQL, enum de status, UUID e relações | FDD |
| C-04 | `src/middlewares/auth.middleware.ts` — `authenticate`, `requireRole` | JWT carrega id/email/role; não customer_id; guard de role já existe | RFC, FDD |
| C-05 | `src/shared/errors/app-error.ts` — `AppError` | Código, HTTP status e details no erro | ADR-006, FDD |
| C-06 | `src/shared/errors/http-errors.ts` | Erros específicos herdam AppError | ADR-006, FDD |
| C-07 | `src/middlewares/error.middleware.ts` | Envelope JSON central e requestId | ADR-006, FDD |
| C-08 | `src/shared/logger/index.ts` | Pino com redaction existente | ADR-006, FDD |
| C-09 | `src/config/database.ts` — `createPrismaClient` | Factory de PrismaClient | FDD |
| C-10 | `src/routes/index.ts` — `buildApiRouter` | Rotas montadas por módulo | FDD |
| C-11 | `src/modules/orders/order.schemas.ts` | Zod valida inputs de pedido | ADR-006, FDD |
| C-12 | `src/modules/orders/order.controller.ts` | Respostas HTTP seguem padrão do projeto | FDD |
| C-13 | `src/server.ts` — `bootstrap` | Entry point atual da API e shutdown do Prisma | ADR-002, FDD |
| C-14 | `package.json` | Stack Node/TypeScript/Prisma/Express/Zod/Pino | FDD |

## Matriz de cobertura preliminar

| Documento | Evidências primárias |
|---|---|
| PRD | T-01 a T-03, T-20 a T-23, T-28 a T-36, T-38, T-42 a T-43 |
| RFC | T-01 a T-19, T-23 a T-24, T-29, T-35, T-42; C-01, C-04 |
| ADRs | T-04 a T-27, T-37, T-44 a T-45; C-01, C-05 a C-08, C-13 |
| FDD | T-05, T-07, T-09 a T-13, T-16 a T-23, T-28 a T-41, T-44 a T-45; C-01 a C-14 |
| Tracker | T-01 a T-45; C-01 a C-14 |

## Lacunas preservadas

1. “Cinco tentativas” e cinco intervalos não definem inequivocamente a contagem do envio inicial. [09:17] Larissa.
2. O JWT não associa usuário a cliente; a reunião não fechou a autorização entre clientes. [09:32] Larissa; `src/middlewares/auth.middleware.ts`.
3. Rate limiting fica para observação; múltiplos workers/particionamento ficam para futuro. [09:39] Larissa; [09:13] Diego.
4. A janela de 24 h de duas secrets não especifica o protocolo de assinatura durante a rotação. [09:21] Sofia.
5. A reunião não especificou status HTTP de sucesso, formato textual da assinatura ou formato de paginação; exemplos no FDD são propostas.
