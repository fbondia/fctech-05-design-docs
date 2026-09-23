# PRD — Sistema de webhooks de notificação de pedidos

Fontes dos requisitos e decisões: [Tracker de rastreabilidade](TRACKER.md).

## Resumo e contexto

Clientes B2B querem receber avisos quando pedidos mudam de status, sem consultar `GET /orders` continuamente. A primeira versão oferece webhooks outbound por API. Atlas Comercial, MaxDistribuição e Nova Cargo fizeram o pedido; Atlas mencionou risco de migração se não houver entrega até o fim do trimestre.

## Problema e motivação

O polling do cliente torna a integração lenta e cara. Para esses clientes, notificação em menos de 10 s atende à expectativa de “tempo real”.

## Público-alvo e cenários de uso

- Equipes de integração cadastram endpoints HTTPS e recebem somente os status escolhidos.
- Usuários autenticados consultam histórico para diagnosticar entregas.
- Administradores reprocessam eventos esgotados.

## Objetivos e métricas de sucesso

| Objetivo | Métrica e meta |
|---|---|
| Notificação rápida | Tempo entre commit e primeira tentativa **< 10 s** em operação saudável; medir distribuição e proporção dentro da meta |
| Integridade | Nenhuma transição confirmada com inscrição aplicável sem evento na outbox |
| Recuperação | Falhas esgotadas registradas na DLQ e passíveis de replay |

A reunião não fixou meta percentual de sucesso; não se presume um SLO adicional.

## Escopo

### Incluso

CRUD autenticado, filtro por status, rotação de secret, entrega assíncrona assinada, histórico de entregas e replay administrativo.

### Fora de escopo

- Email ao cliente sobre falhas; próxima fase.
- Dashboard visual; nesta fase somente API.
- Rate limiting de saída; observar e decidir depois.
- Arquivamento das linhas entregues após cerca de 30 dias.
- Múltiplos workers e ordenação global.

## Requisitos funcionais

| ID | Requisito |
|---|---|
| PRD-FR-01 | Cadastrar endpoint com URL, secret gerada e status desejados. |
| PRD-FR-02 | Editar, remover e listar endpoints do cliente. |
| PRD-FR-03 | Filtrar na inserção: sem inscrição no status, não criar evento. |
| PRD-FR-04 | Registrar evento na transação da mudança de status. |
| PRD-FR-05 | Entregar evento assinado com ID estável para deduplicação. |
| PRD-FR-06 | Retentar com backoff finito e guardar falha definitiva em DLQ. |
| PRD-FR-07 | Replay manual só por `ADMIN`, com auditoria de autor. |
| PRD-FR-08 | Histórico recente com sucesso/falha, payload, resposta e duração. |
| PRD-FR-09 | Rotacionar secret; anterior válida por 24 h. |

## Requisitos não funcionais

- PRD-NFR-01: primeira tentativa em menos de 10 s em operação saudável; polling a cada 2 s.
- PRD-NFR-02: HTTPS, HMAC-SHA256 por endpoint, limite de 64 KB com erro acima do teto.
- PRD-NFR-03: timeout HTTP de 10 s.
- PRD-NFR-04: at-least-once; ordem por pedido enquanto houver um worker.

## Decisões e trade-offs principais

Outbox no MySQL evita HTTP na transação e Redis adicional, ao custo de escrita e polling. Retry finito exige DLQ operacional. At-least-once requer deduplicação pelo cliente. Ver [RFC](RFC.md) e [ADRs](adrs/README.md).

## Dependências

MySQL/Prisma e transação do pedido (`prisma/schema.prisma`, `src/modules/orders/order.service.ts`), JWT e role `ADMIN` (`src/middlewares/auth.middleware.ts`), revisão de Sofia por dois dias úteis antes do deploy.

## Riscos e mitigação

Probabilidade e impacto são **avaliações de planejamento**, não estimativas da reunião.

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Acúmulo da outbox | Média | Alto | Índices, lotes pequenos e monitorar idade da pendência. |
| Destino indisponível | Alta | Médio | Backoff finito, DLQ e replay. |
| Vazamento de secret | Média | Alto | Secret individual, rotação e revisão de segurança. |

## Critérios de aceitação

1. Transição válida com inscrição confirma pedido, histórico e evento juntos; rollback não deixa evento órfão.
2. Destino saudável recebe evento em menos de 10 s com assinatura e headers, sem bloquear mudança do pedido.
3. Cliente configura filtro, consulta histórico e rotaciona secret; replay exige `ADMIN` e auditoria.
4. Falhas esgotadas aparecem na DLQ com payload e motivo; replay cria nova pendência.

## Estratégia de testes e validação

Validar transação/rollback, filtros, HMAC/rotação/HTTPS/tamanho, timeout e falhas até DLQ, replay com e sem `ADMIN`, duplicata com mesmo `X-Event-Id` e latência saudável. Fechar autorização por `customer_id` e contagem de tentativas na revisão do [RFC](RFC.md) antes dos testes finais.
