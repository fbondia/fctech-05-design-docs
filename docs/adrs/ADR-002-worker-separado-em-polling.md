# ADR-002 — Worker separado com polling

**Evidências do [inventário](../EVIDENCIAS.md):** T-07, T-09, T-10, T-11, T-12, T-13; C-09, C-13.

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
O envio não deve bloquear a API. O MySQL usado no projeto não oferece um mecanismo nativo de notificação ao processo externo para este fluxo.

## Decisão
Executar um único worker em processo separado (entry point planejado `src/worker.ts`), com PrismaClient próprio apontando para o mesmo banco. Consultar pendências a cada 2 segundos, em lotes pequenos, por `created_at`. A reunião limita a ordenação por pedido ao cenário de um worker; não há garantia global. O FDD detalha o bloqueio necessário durante retry para cumprir essa intenção.

## Alternativas consideradas
- Trigger SQL como aviso ao processo: não notifica o worker diretamente.
- Worker dentro da API: reinícios da API interrompem o processamento.

## Consequências
- Positiva: solução simples, compatível com o alvo de menos de 10 segundos em operação saudável.
- Negativa: polling gera leituras periódicas; escalar para múltiplos workers requer redesenhar bloqueio/particionamento por `order_id`.
