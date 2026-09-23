# ADR-001 — Outbox transacional no MySQL

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
`OrderService.changeStatus` altera pedido, histórico e estoque em uma transação Prisma (`src/modules/orders/order.service.ts`). Uma chamada HTTP nesse caminho bloquearia a mudança de status e poderia falhar após o commit. [09:04] Bruno; [09:06] Diego.

## Decisão
Inserir um evento com UUID e payload congelado em `webhook_outbox`, na mesma transação SQL da mudança de status. A função `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebe o cliente transacional; falha na inserção causa rollback. Usar o MySQL já disponível. [09:06] Diego; [09:40–09:41] Bruno; [09:51–09:52] Larissa.

## Alternativas consideradas
- HTTP síncrono dentro do serviço: bloqueia a transação se o destino estiver lento ou indisponível. [09:03–09:06] Larissa, Bruno e Diego.
- Redis Streams: exigiria infraestrutura adicional para o time pequeno. [09:07] Larissa e Diego.

## Consequências
- Positiva: mudança de status e registro do evento são atômicos.
- Negativa: há escrita adicional na transação e operação de uma tabela de outbox; a entrega continua assíncrona e sujeita a falhas posteriores.

