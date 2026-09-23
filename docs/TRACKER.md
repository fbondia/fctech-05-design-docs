# Tracker de rastreabilidade

Cada linha representa um requisito, decisão, restrição, alternativa ou integração identificável. IDs de PRD são iguais aos do documento; os demais IDs identificam seções/itens deste tracker. Propostas de formato HTTP, estrutura de colunas e instrumentação no FDD são rotuladas como propostas de implementação e derivam das fontes listadas; não são apresentadas como decisões fechadas.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CTX-01 | docs/PRD.md | Problema | Três clientes B2B pedem aviso de mudança de status | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Meta | Menos de 10 s é aceitável | TRANSCRICAO | [09:02] Marcos |
| PRD-CTX-03 | docs/PRD.md | Restrição | Apenas webhooks outbound | TRANSCRICAO | [09:02] Marcos |
| PRD-GOAL-01 | docs/PRD.md | Métrica | Primeira tentativa em menos de 10 s | TRANSCRICAO | [09:02] Marcos |
| PRD-GOAL-02 | docs/PRD.md | Métrica | Status confirmado deve ter evento atômico quando há inscrição | TRANSCRICAO | [09:06] Diego |
| PRD-GOAL-03 | docs/PRD.md | Métrica | Falha definitiva recuperável pela DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Criar endpoint com URL, secret gerada e filtros | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Editar, remover e listar endpoints | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Filtrar status antes da inserção na outbox | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Registrar evento na transação do pedido | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Enviar assinatura e ID estável | TRANSCRICAO | [09:25] Diego |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Retry finito e DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Replay só por ADMIN, com auditoria | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Rotação de secret com 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Polling de 2 s para meta <10 s | TRANSCRICAO | [09:09] Diego |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | HTTPS, HMAC e 64 KB | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 s | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | At-least-once e ordem limitada a um worker | TRANSCRICAO | [09:12] Diego |
| PRD-EX-01 | docs/PRD.md | Exclusão | Email futuro | TRANSCRICAO | [09:37] Larissa |
| PRD-EX-02 | docs/PRD.md | Exclusão | Dashboard visual futuro | TRANSCRICAO | [09:40] Larissa |
| PRD-EX-03 | docs/PRD.md | Exclusão | Rate limiting adiado | TRANSCRICAO | [09:39] Larissa |
| PRD-EX-04 | docs/PRD.md | Exclusão | Arquivamento fora de escopo | TRANSCRICAO | [09:08] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-RISK-01 | docs/PRD.md | Risco | Acúmulo na outbox; mitigar com índice e lote pequeno | TRANSCRICAO | [09:08] Diego |
| PRD-RISK-02 | docs/PRD.md | Risco | Destino indisponível; mitigar com retry e DLQ | TRANSCRICAO | [09:18] Diego |
| PRD-RISK-03 | docs/PRD.md | Risco | Secret vazada; mitigar com secret individual e rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-01 | docs/PRD.md | Aceite | Rollback conjunto de pedido e outbox | TRANSCRICAO | [09:41] Bruno |
| PRD-AC-02 | docs/PRD.md | Aceite | Entrega sem bloquear o pedido | TRANSCRICAO | [09:04] Bruno |
| PRD-AC-03 | docs/PRD.md | Aceite | Configuração, histórico e replay controlado | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-04 | docs/PRD.md | Aceite | Falha definitiva em DLQ com replay | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-01 | docs/RFC.md | Proposta | Configuração e filtro por cliente | TRANSCRICAO | [09:34] Bruno |
| RFC-PROP-02 | docs/RFC.md | Proposta | Worker único separado em polling 2 s | TRANSCRICAO | [09:11] Diego |
| RFC-PROP-03 | docs/RFC.md | Proposta | Retry, DLQ, histórico e replay | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-04 | docs/RFC.md | Proposta | Reuso da stack existente | TRANSCRICAO | [09:30] Larissa |
| RFC-IMPACT-01 | docs/RFC.md | Impacto | Escrita extra em changeStatus e novo processo | TRANSCRICAO | [09:41] Bruno |
| RFC-IMPACT-02 | docs/RFC.md | Risco | Acúmulo da outbox eleva latência | TRANSCRICAO | [09:07] Bruno |
| RFC-ALT-01 | docs/RFC.md | Alternativa | HTTP síncrono bloqueia transação | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa | Redis exigiria infraestrutura | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa | Trigger não notifica worker externo | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa | Exactly-once exigiria coordenação | TRANSCRICAO | [09:25] Diego |
| RFC-OPEN-01 | docs/RFC.md | Questão em aberto | Cinco tentativas versus cinco intervalos | TRANSCRICAO | [09:17] Diego |
| RFC-OPEN-02 | docs/RFC.md | Questão em aberto | Customer ID não é implícito do JWT | TRANSCRICAO | [09:32] Bruno |
| RFC-OPEN-03 | docs/RFC.md | Questão em aberto | Regra usuário→cliente não existe no JWT | CODIGO | src/middlewares/auth.middleware.ts |
| RFC-OPEN-04 | docs/RFC.md | Questão em aberto | Rate limiting a observar | TRANSCRICAO | [09:39] Diego |
| RFC-OPEN-05 | docs/RFC.md | Questão em aberto | Escala futura por order_id | TRANSCRICAO | [09:13] Diego |
| RFC-PLAN-01 | docs/RFC.md | Planejamento | Três sprints e revisão de Sofia | TRANSCRICAO | [09:46] Larissa |
| FDD-DATA-01 | docs/FDD.md | Modelo | Configuração guarda URL, secret, customer e ativo | TRANSCRICAO | [09:21] Sofia |
| FDD-DATA-02 | docs/FDD.md | Modelo | Outbox indexada por status e criação | TRANSCRICAO | [09:08] Diego |
| FDD-DATA-03 | docs/FDD.md | Modelo | DLQ separada com payload e motivo | TRANSCRICAO | [09:18] Diego |
| FDD-DATA-04 | docs/FDD.md | Modelo | UUID é padrão para ID da outbox | TRANSCRICAO | [09:51] Larissa |
| FDD-FLOW-01 | docs/FDD.md | Fluxo | Função publishWebhookEvent recebe tx | TRANSCRICAO | [09:41] Bruno |
| FDD-FLOW-02 | docs/FDD.md | Fluxo | Snapshot na inserção | TRANSCRICAO | [09:52] Larissa |
| FDD-FLOW-03 | docs/FDD.md | Fluxo | Worker separado e PrismaClient próprio | TRANSCRICAO | [09:30] Bruno |
| FDD-FLOW-03A | docs/FDD.md | Proposta técnica | Bloquear evento posterior do mesmo pedido enquanto anterior aguarda retry, para cumprir ordering | TRANSCRICAO | [09:12] Diego |
| FDD-FLOW-03B | docs/FDD.md | Proposta técnica | Claim atômico e recuperação após reinício para sustentar at-least-once | TRANSCRICAO | [09:24] Diego |
| FDD-FLOW-04 | docs/FDD.md | Fluxo | Retry 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego |
| FDD-FLOW-05 | docs/FDD.md | Fluxo | Replay recoloca evento pendente | TRANSCRICAO | [09:18] Diego |
| FDD-RES-01 | docs/FDD.md | Resiliência | Timeout HTTP de 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Resiliência | Falha do destino não reverte status | TRANSCRICAO | [09:04] Bruno |
| FDD-RES-03 | docs/FDD.md | Resiliência | Mesmo X-Event-Id em reenvios | TRANSCRICAO | [09:25] Diego |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST cadastra endpoint, secret gerada | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET/PATCH/DELETE configurações | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | Endpoint de rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | GET deliveries com últimos envios | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | POST admin replay DLQ | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | Payload JSON sem items | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | Headers de entrega | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | X-Webhook-Id adicional | TRANSCRICAO | [09:44] Sofia |
| FDD-ERR-01 | docs/FDD.md | Erro | Prefixo WEBHOOK_ | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Erro | WEBHOOK_INVALID_URL aplica HTTPS | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-03 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND segue erro tipado | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERR-04 | docs/FDD.md | Erro | WEBHOOK_INVALID_STATUS_FILTER valida enum | CODIGO | prisma/schema.prisma |
| FDD-ERR-05 | docs/FDD.md | Erro | WEBHOOK_PAYLOAD_TOO_LARGE para >64 KB | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-06 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_TIMEOUT após 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-07 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_FAILED para entrega falha | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-08 | docs/FDD.md | Erro | WEBHOOK_DEAD_LETTER_NOT_FOUND no replay | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-09 | docs/FDD.md | Erro | WEBHOOK_CUSTOMER_ACCESS_DENIED depende de autorização pendente | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Reusar Pino | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Logger existente usa Pino | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | Middleware registra requestId | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus abre transação Prisma | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Máquina de estados centralizada | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | MySQL, UUID e OrderStatus existentes | CODIGO | prisma/schema.prisma |
| FDD-INT-04 | docs/FDD.md | Integração | authenticate e requireRole disponíveis | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-05 | docs/FDD.md | Integração | AppError fornece código e status | CODIGO | src/shared/errors/app-error.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Envelope de erro central | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-07 | docs/FDD.md | Integração | API monta rotas por módulo | CODIGO | src/routes/index.ts |
| FDD-INT-08 | docs/FDD.md | Integração | PrismaClient por processo | CODIGO | src/config/database.ts |
| FDD-AC-01 | docs/FDD.md | Aceite técnico | Teste transacional e filtro sem inscrição | TRANSCRICAO | [09:41] Bruno |
| FDD-AC-02 | docs/FDD.md | Aceite técnico | Timeout de entrega de 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-AC-03 | docs/FDD.md | Aceite técnico | HMAC, rotação, HTTPS e 64 KB | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-04 | docs/FDD.md | Aceite técnico | DLQ/replay com ADMIN auditado | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-05 | docs/FDD.md | Aceite técnico | X-Event-Id estável e um worker | TRANSCRICAO | [09:25] Diego |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox atômica no MySQL | TRANSCRICAO | [09:06] Diego |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Processo separado com polling 2 s | TRANSCRICAO | [09:11] Diego |
| ADR-003 | docs/adrs/ADR-003-retry-e-dead-letter.md | Decisão | Retry finito e DLQ separada | TRANSCRICAO | [09:18] Diego |
| ADR-004 | docs/adrs/ADR-004-hmac-e-rotacao-de-secret.md | Decisão | HMAC-SHA256 individual e rotação | TRANSCRICAO | [09:22] Sofia |
| ADR-005 | docs/adrs/ADR-005-at-least-once-e-event-id.md | Decisão | At-least-once com X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reusar padrões existentes | TRANSCRICAO | [09:30] Larissa |

## Auditoria de cobertura

Unidade de contagem: item normativo distinto (requisito, meta, exclusão, decisão, alternativa, questão aberta, fluxo, contrato, erro, integração, risco ou aceite) nas tabelas/listas e seções dos documentos. Explicações que repetem o mesmo item em outra seção não são contadas novamente. Os IDs acima identificam o universo auditado e tornam a contagem reproduzível.

| Documento | Itens identificáveis | Itens rastreados | Cobertura |
|---|---:|---:|---:|
| PRD | 31 | 31 | 100% |
| RFC | 16 | 16 | 100% |
| FDD | 47 | 47 | 100% |
| ADRs | 6 | 6 | 100% |
| **Total** | **100** | **100** | **100%** |

Das 100 linhas, 86 usam `TRANSCRICAO` com timestamp e falante (86%) e 14 usam `CODIGO` com caminho real. O limite pedido é 80% de cobertura, 70% de linhas da transcrição e pelo menos 5 linhas de código. A contagem cobre os itens identificados segundo a unidade acima; decisões ainda em aberto continuam rotuladas nos documentos.
