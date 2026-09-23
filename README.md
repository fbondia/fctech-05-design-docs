# Design docs — Webhooks de notificação de pedidos

## Sobre o desafio

Este repositório é o fork do OMS usado no desafio 05. A entrega transforma a [transcrição da reunião](TRANSCRICAO.md) e o código existente em uma proposta documental para implementar webhooks de mudança de status de pedidos. Nenhum arquivo de `src/`, `prisma/`, `tests/` ou configuração da aplicação foi alterado.

Os documentos separam problema de produto, proposta arquitetural, decisões pontuais e detalhes de implementação. O [tracker](docs/TRACKER.md) liga requisitos, decisões e restrições às falas ou a arquivos reais do projeto. Trechos não fechados na reunião são identificados como proposta ou questão em aberto.

## Ferramentas de IA utilizadas

- **Codex (GPT-6)**: leitura dirigida da transcrição e do código, redação dos documentos, revisão de consistência e checagens locais.
- **Ferramentas de terminal do Codex**: buscas com `rg`, inspeção do Git e validação de caminhos, links e cobertura; são ferramentas de apoio ao trabalho com IA, não uma fonte de requisitos.

## Workflow adotado

1. Clonar o fork e ler o enunciado e a transcrição integralmente.
2. Inspecionar `OrderService.changeStatus`, schema Prisma, autenticação, erros, rotas e logger para distinguir fatos do código de intenções da reunião. Registrar as fontes em [EVIDENCIAS.md](docs/EVIDENCIAS.md).
3. Classificar falas como decisões, requisitos, alternativas descartadas, exclusões e lacunas. Criar primeiro os seis ADRs.
4. Consolidar o RFC em nível de arquitetura; elaborar o PRD em nível de produto e o FDD com fluxos e contratos propostos.
5. Construir o tracker com origem para cada grupo de itens e revisar coerência entre os documentos. O README registra o processo após essa revisão.

## Prompts customizados

Os prompts abaixo resumem as instruções dirigidas usadas na produção; os caminhos e timestamps fazem parte do controle de rastreabilidade.

```text
Leia TRANSCRICAO.md e src/modules/orders/order.service.ts. Extraia decisões fechadas,
requisitos, alternativas descartadas, itens adiados e ambiguidades em grupos separados.
Para cada afirmação, forneça [hh:mm] falante ou arquivo real. Não transforme
alternativas ou sugestões em requisito; marque lacunas sem completá-las por suposição.
```

```text
Revise PRD, RFC, FDD e ADRs contra TRANSCRICAO.md e o código. Procure
contradições, caminhos inexistentes, duplicação de nível entre RFC/FDD e
contratos apresentados como decisões sem base. Verifique em especial JWT sem
customer_id e a contagem de cinco tentativas versus cinco intervalos de retry.
Corrija os documentos e atualize TRACKER.md com a origem de cada item.
```

## Iterações e ajustes

Foram **três ciclos principais**: (1) extração e ADRs; (2) RFC/PRD/FDD; (3) revisão cruzada e rastreabilidade. Os ajustes concretos mais importantes foram:

- A primeira leitura poderia tratar `customer_id` como implícito no JWT. O código em `src/middlewares/auth.middleware.ts` e a correção de Larissa na reunião mostram que o token só carrega usuário e role. O RFC e o FDD passaram a expor a regra de autorização por cliente como pendente.
- A política discutida cita “5 tentativas” junto de cinco intervalos. Em vez de codificar silenciosamente seis envios ou descartar um intervalo, o ADR-003, RFC e FDD registram a ambiguidade e pedem confirmação antes da implementação.
- Os exemplos HTTP exigidos pelo FDD usam caminhos e status propostos quando a reunião decidiu a operação, mas não o formato exato. Eles foram rotulados como proposta para não virar falso requisito.

## Como navegar a entrega

1. [PRD](docs/PRD.md): problema, público, escopo e aceite.
2. [RFC](docs/RFC.md): arquitetura, alternativas e questões em aberto.
3. [ADRs](docs/adrs/README.md): seis decisões isoladas e seus trade-offs.
4. [FDD](docs/FDD.md): fluxos, contratos, erros e integração com o código.
5. [Tracker](docs/TRACKER.md): origem de cada item identificável.
6. [Auditoria](docs/AUDITORIA.md): checklist final, contagens e limites da entrega.

A reunião original permanece em [TRANSCRICAO.md](TRANSCRICAO.md).
