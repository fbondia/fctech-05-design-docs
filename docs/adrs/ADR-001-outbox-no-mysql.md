# ADR-001 — Outbox transacional no MySQL

**Evidências do [inventário](../EVIDENCIAS.md):** T-04, T-05, T-06, T-37, T-44, T-45; C-01, C-03.

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
`OrderService.changeStatus` altera pedido, histórico e estoque em uma transação Prisma (`src/modules/orders/order.service.ts`). Uma chamada HTTP nesse caminho bloquearia a mudança de status e poderia falhar após o commit.

## Decisão
Inserir um evento com UUID e payload congelado em `webhook_outbox`, na mesma transação SQL da mudança de status. A função `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebe o cliente transacional; falha na inserção causa rollback. Usar o MySQL já disponível.

## Alternativas consideradas
- HTTP síncrono dentro do serviço: bloqueia a transação se o destino estiver lento ou indisponível.
- Redis Streams: exigiria infraestrutura adicional para o time pequeno.

## Consequências
- Positiva: mudança de status e registro do evento são atômicos.
- Negativa: há escrita adicional na transação e operação de uma tabela de outbox; a entrega continua assíncrona e sujeita a falhas posteriores.
