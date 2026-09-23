# RFC — Webhooks de mudanças de status de pedidos

Fontes das decisões, alternativas e questões abertas: [Tracker de rastreabilidade](TRACKER.md).

| Campo | Valor |
|---|---|
| Autor | Equipe de engenharia; documento elaborado com IA a partir da reunião |
| Status | Proposta para revisão |
| Data | 2026-09-23 (elaboração; data da reunião não informada) |
| Revisores | Larissa, Marcos, Bruno, Diego e Sofia |

## TL;DR

Registrar um snapshot de evento na mesma transação da mudança de status e entregá-lo via worker separado em polling de 2 s. Assinar cada envio com HMAC-SHA256 por endpoint, retentar falhas com backoff e mover falhas definitivas para DLQ. O consumidor deduplica por `X-Event-Id`.

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo consultam `GET /orders` repetidamente e pedem notificação em menos de 10 s. O fluxo é só outbound. `changeStatus` já atualiza pedido, histórico e estoque na transação (`src/modules/orders/order.service.ts`); HTTP nesse caminho acoplaria a operação ao destino.

## Proposta técnica

1. Persistir configuração por `customer_id`, URL HTTPS, secret individual, estado ativo e filtro de status. Para cada transição, filtrar inscrições e inserir um snapshot por endpoint na transação do pedido.
2. Um único worker separado consulta pendências antigas a cada 2 s em lotes pequenos. Envia JSON com `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id`; timeout 10 s e limite 64 KB.
3. Retry finito e dead letter persistida, com replay administrativo. Histórico de entregas disponível ao cliente.
4. Reusar Express, Prisma, Zod, Pino, `AppError` e o padrão de módulos.

Contratos e integração ficam no [FDD](FDD.md).

## Alternativas consideradas

| Alternativa | Trade-off do descarte |
|---|---|
| HTTP síncrono no `OrderService` | Destino lento ou indisponível bloquearia a transação. |
| Redis Streams | Acrescentaria infraestrutura para um time pequeno. |
| Trigger SQL para despertar worker | Não notifica diretamente processo externo no MySQL. |
| Exactly-once distribuído | Exigiria coordenação entre produtor e consumidor. |

## Questões em aberto

1. **Contagem de retry:** “5 tentativas” e cinco intervalos de backoff aparecem juntos. Confirmar se o envio inicial está incluído.
2. **Autorização por cliente:** o `customer_id` virá do body ou path; o JWT atual representa usuário, não cliente. A reunião não definiu como restringir acesso entre clientes. O token atual pode ser consultado em `src/middlewares/auth.middleware.ts`.
3. **Rate limiting de saída:** observar e decidir em outra fase.
4. **Escala/ordenação:** particionamento por `order_id` ou lock apenas se houver múltiplos workers.

## Impacto e riscos

Novas tabelas de configuração, outbox, tentativas e DLQ; escrita extra em `changeStatus`; processo separado. Falha da inserção na outbox deve abortar a transição. Acúmulo de pendências pode elevar latência; índice por estado/criação, lotes pequenos e métricas reduzem o risco. Estimativa: três sprints, com dois dias úteis de revisão de segurança antes do deploy.

## Decisões relacionadas

[ADR-001](adrs/ADR-001-outbox-no-mysql.md) · [ADR-002](adrs/ADR-002-worker-separado-em-polling.md) · [ADR-003](adrs/ADR-003-retry-e-dead-letter.md) · [ADR-004](adrs/ADR-004-hmac-e-rotacao-de-secret.md) · [ADR-005](adrs/ADR-005-at-least-once-e-event-id.md) · [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
