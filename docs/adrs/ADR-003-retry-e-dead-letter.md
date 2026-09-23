# ADR-003 — Retry com backoff e dead letter

**Evidências do [inventário](../EVIDENCIAS.md):** T-14, T-15, T-16, T-17, T-18.

## Status
Decidido na reunião; semântica da contagem de tentativas pendente de confirmação.

## Contexto
Destinos externos podem ficar indisponíveis por horas. Três tentativas foram consideradas insuficientes; retry ilimitado deixa eventos presos indefinidamente.

## Decisão
Usar backoff de 1 min, 5 min, 30 min, 2 h e 12 h e limite anunciado de cinco tentativas. Após esgotar a política, persistir payload, falha e horário em `webhook_dead_letter`; replay manual por endpoint administrativo.

**Ponto de revisão:** cinco intervalos implicam cinco retries após a primeira tentativa, enquanto “cinco tentativas” pode significar cinco envios totais. O RFC mantém essa ambiguidade aberta; a implementação deve fixar a convenção antes de codificar o agendamento.

## Alternativas consideradas
- Três tentativas: janela curta para indisponibilidade planejada de duas horas.
- Retry indefinido: evento pode ficar pendurado para sempre.
- Marcar `failed` na própria outbox: poluiria a consulta de pendências; a tabela separada favorece diagnóstico e replay.

## Consequências
- Positiva: falhas transitórias recebem nova chance e falhas permanentes ficam visíveis.
- Negativa: entrega pode atrasar por horas; há armazenamento e operação de replay a manter.
