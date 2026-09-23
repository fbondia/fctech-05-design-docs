# ADR-005 — Entrega at-least-once com X-Event-Id

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
O worker pode reenviar um evento quando uma resposta de sucesso se perde ou após falha temporária. [09:24–09:26] Diego.

## Decisão
Garantir entrega at-least-once e enviar o UUID criado na outbox em `X-Event-Id`. O consumidor deduplica pelo identificador. [09:25–09:26] Diego e Larissa.

## Alternativas consideradas
- Exactly-once entre dois sistemas: exigiria coordenação entre produtor e consumidor e foi descartado pela complexidade. [09:25] Diego.

## Consequências
- Positiva: retry não depende de protocolo distribuído de confirmação.
- Negativa: o cliente precisa suportar duplicatas e persistir os IDs processados.

