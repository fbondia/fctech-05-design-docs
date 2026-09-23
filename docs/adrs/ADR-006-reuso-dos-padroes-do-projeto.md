# ADR-006 — Reuso dos padrões do projeto

**Evidências do [inventário](../EVIDENCIAS.md):** T-25, T-26, T-27; C-04 a C-11.

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
O projeto organiza domínios em controller, service, repository, routes e schemas (`src/modules/orders/`). Usa Zod (`src/modules/orders/order.schemas.ts`), `AppError` (`src/shared/errors/app-error.ts`), middleware central (`src/middlewares/error.middleware.ts`) e Pino (`src/shared/logger/index.ts`).

## Decisão
Criar o módulo planejado `src/modules/webhooks` no mesmo padrão, com processador do worker no módulo e entry point planejado `src/worker.ts`. Reusar Zod, `AppError`, middleware, Pino e autenticação existente. Erros específicos do domínio usam prefixo `WEBHOOK_`. Cada processo cria seu próprio PrismaClient.

## Alternativas consideradas
- Introduzir stack ou tratamento de erros próprio: duplicaria padrões já disponíveis e foi afastado pelo objetivo explícito de reuso.

## Consequências
- Positiva: integração e operação seguem convenções já conhecidas.
- Negativa: o módulo herda limites do JWT atual, que só carrega usuário e role (`src/middlewares/auth.middleware.ts`); o vínculo do usuário ao `customer_id` precisa ser definido antes da implementação de autorização por cliente.
