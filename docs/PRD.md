# PRD — Sistema de webhooks de notificação de pedidos

## Resumo e contexto

Clientes B2B querem receber avisos quando pedidos mudam de status, sem consultar `GET /orders` continuamente. A primeira versão oferece webhooks outbound por API. Atlas Comercial, MaxDistribuição e Nova Cargo fizeram o pedido; Atlas mencionou risco de migração se não houver entrega até o fim do trimestre. [09:00–09:03] Marcos e Sofia.

## Problema e motivação

O polling do cliente torna a integração lenta e cara. Para esses clientes, notificação em menos de 10 s atende à expectativa de “tempo real”. [09:00–09:02] Marcos.

## Público-alvo e cenários de uso

- Equipes de integração cadastram endpoints HTTPS e recebem somente os status escolhidos. [09:31–09:34] Marcos e Bruno.
- Usuários autenticados consultam histórico para diagnosticar entregas. [09:34–09:37] Marcos e Sofia.
- Administradores reprocessam eventos esgotados. [09:18–09:19] Diego e Larissa; [09:35–09:36] Sofia.

## Objetivos e métricas de sucesso

| Objetivo | Métrica e meta | Fonte |
|---|---|---|
| Notificação rápida | Tempo entre commit e primeira tentativa **< 10 s** em operação saudável; medir distribuição e proporção dentro da meta | [09:02] Marcos; [09:09–09:10] Diego e Marcos |
| Integridade | Nenhuma transição confirmada com inscrição aplicável sem evento na outbox | [09:06] Diego; [09:40–09:41] Bruno |
| Recuperação | Falhas esgotadas registradas na DLQ e passíveis de replay | [09:17–09:19] Diego |

A reunião não fixou meta percentual de sucesso; não se presume um SLO adicional.

## Escopo

### Incluso

CRUD autenticado, filtro por status, rotação de secret, entrega assíncrona assinada, histórico de entregas e replay administrativo. [09:21–09:22] Sofia; [09:31–09:36] Marcos, Bruno e Diego.

### Fora de escopo

- Email ao cliente sobre falhas; próxima fase. [09:37–09:38] Marcos e Larissa.
- Dashboard visual; nesta fase somente API. [09:39–09:40] Marcos e Larissa.
- Rate limiting de saída; observar e decidir depois. [09:38–09:39] Diego e Larissa.
- Arquivamento das linhas entregues após cerca de 30 dias. [09:08] Diego.
- Múltiplos workers e ordenação global. [09:12–09:14] Diego e Larissa.

## Requisitos funcionais

| ID | Requisito | Fonte |
|---|---|---|
| PRD-FR-01 | Cadastrar endpoint com URL, secret gerada e status desejados. | [09:31–09:33] Marcos e Larissa |
| PRD-FR-02 | Editar, remover e listar endpoints do cliente. | [09:33] Bruno |
| PRD-FR-03 | Filtrar na inserção: sem inscrição no status, não criar evento. | [09:33–09:34] Marcos e Bruno |
| PRD-FR-04 | Registrar evento na transação da mudança de status. | [09:06] Diego; [09:40–09:41] Bruno |
| PRD-FR-05 | Entregar evento assinado com ID estável para deduplicação. | [09:20–09:26] Sofia e Diego |
| PRD-FR-06 | Retentar com backoff finito e guardar falha definitiva em DLQ. | [09:15–09:19] Diego e Larissa |
| PRD-FR-07 | Replay manual só por `ADMIN`, com auditoria de autor. | [09:18–09:19] Diego; [09:35–09:36] Sofia |
| PRD-FR-08 | Histórico recente com sucesso/falha, payload, resposta e duração. | [09:34] Marcos |
| PRD-FR-09 | Rotacionar secret; anterior válida por 24 h. | [09:21–09:22] Sofia |

## Requisitos não funcionais

- PRD-NFR-01: primeira tentativa em menos de 10 s em operação saudável; polling a cada 2 s. [09:02] Marcos; [09:09–09:10] Diego.
- PRD-NFR-02: HTTPS, HMAC-SHA256 por endpoint, limite de 64 KB com erro acima do teto. [09:20–09:24] Sofia e Diego.
- PRD-NFR-03: timeout HTTP de 10 s. [09:42] Diego.
- PRD-NFR-04: at-least-once; ordem por pedido enquanto houver um worker. [09:12–09:13] Diego e Larissa; [09:24–09:26] Diego.

## Decisões e trade-offs principais

Outbox no MySQL evita HTTP na transação e Redis adicional, ao custo de escrita e polling. Retry finito exige DLQ operacional. At-least-once requer deduplicação pelo cliente. Ver [RFC](RFC.md) e [ADRs](adrs/README.md). [09:04–09:07] Bruno e Diego; [09:15–09:18] Diego; [09:24–09:26] Diego.

## Dependências

MySQL/Prisma e transação do pedido (`prisma/schema.prisma`, `src/modules/orders/order.service.ts`), JWT e role `ADMIN` (`src/middlewares/auth.middleware.ts`), revisão de Sofia por dois dias úteis antes do deploy. [09:29–09:30] Bruno; [09:36] Sofia; [09:46] Sofia.

## Riscos e mitigação

Probabilidade e impacto são **avaliações de planejamento**, não estimativas da reunião.

| Risco | Probabilidade | Impacto | Mitigação e fonte |
|---|---|---|---|
| Acúmulo da outbox | Média | Alto | Índices, lotes pequenos e monitorar idade da pendência. [09:07–09:08] Bruno e Diego |
| Destino indisponível | Alta | Médio | Backoff finito, DLQ e replay. [09:15–09:19] Diego |
| Vazamento de secret | Média | Alto | Secret individual, rotação e revisão de segurança. [09:21–09:22] Sofia |

## Critérios de aceitação

1. Transição válida com inscrição confirma pedido, histórico e evento juntos; rollback não deixa evento órfão. [09:06] Diego; [09:40–09:41] Bruno.
2. Destino saudável recebe evento em menos de 10 s com assinatura e headers, sem bloquear mudança do pedido. [09:02] Marcos; [09:04] Bruno; [09:44–09:45] Diego e Sofia.
3. Cliente configura filtro, consulta histórico e rotaciona secret; replay exige `ADMIN` e auditoria. [09:21–09:22] Sofia; [09:31–09:36] Marcos, Bruno e Sofia.
4. Falhas esgotadas aparecem na DLQ com payload e motivo; replay cria nova pendência. [09:17–09:19] Diego.

## Estratégia de testes e validação

Validar transação/rollback, filtros, HMAC/rotação/HTTPS/tamanho, timeout e falhas até DLQ, replay com e sem `ADMIN`, duplicata com mesmo `X-Event-Id` e latência saudável. Fechar autorização por `customer_id` e contagem de tentativas na revisão do [RFC](RFC.md) antes dos testes finais.
