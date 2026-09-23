# Auditoria final — desafio 05

**Fork:** https://github.com/fbondia/fctech-05-design-docs  
**Baseline do fork:** `e7f6311` (`main` no momento do clone)  
**Escopo:** documentação apenas; comparar `git diff --name-only e7f6311..HEAD` e o working tree antes de publicar.

| Critério do enunciado e do Biaws | Evidência verificada | Resultado |
|---|---|---|
| Fork público e fontes | URL acessível anonimamente por HTTP 200; `TRANSCRICAO.md` presente; baseline acima | Atende |
| Inventário prévio de fontes | [EVIDENCIAS.md](EVIDENCIAS.md): 45 falas e 14 entradas de código, com destinos e lacunas | Atende |
| 5–8 ADRs | 6 arquivos `ADR-001` a `ADR-006`; cinco seções exigidas em cada | Atende |
| RFC | [RFC.md](RFC.md): metadados, 4 alternativas, 4 questões abertas e 6 links para ADRs | Atende |
| FDD | [FDD.md](FDD.md): fluxos, 6 endpoints inbound com exemplos, envio outbound, 8 códigos `WEBHOOK_*`, observabilidade e 8 integrações reais | Atende |
| PRD | [PRD.md](PRD.md): 9 RFs, meta <10 s, 4 exclusões, 3 riscos com probabilidade/impacto/mitigação | Atende |
| Tracker | [TRACKER.md](TRACKER.md): 100/100 itens identificados, 86/100 (86%) da transcrição, 14 caminhos de código | Atende |
| README | [README](../README.md): ferramentas, workflow, 2 prompts e 3 ajustes concretos | Atende |
| Integridade do código | Nenhum arquivo `src/`, `prisma/`, `tests/`, configuração ou `TRANSCRICAO.md` alterado desde a baseline | Atende |
| Links e caminhos | Verificação local de links Markdown e de caminhos de código existente; referências a arquivos futuros estão marcadas como planejadas | Atende |
| Git | `git diff --check` sem erro; commit publicado no fork e working tree limpo | Atende |

## Método e limites

A cobertura do tracker conta unidades normativas distintas, conforme a regra explícita no próprio [tracker](TRACKER.md); prosa que repete o mesmo item não aumenta o denominador. Todos os 100 IDs são únicos; timestamps+falantes e 14 caminhos de código foram confrontados com as fontes locais. A data da reunião não está na transcrição, portanto não foi inventada. Probabilidade/impacto dos riscos e convenções HTTP propostas estão identificados como avaliações/propostas, não como decisões da call.

## Pendências de decisão mantidas no RFC

1. Contagem do envio inicial nas “5 tentativas” versus cinco intervalos de backoff.
2. Regra de autorização usuário→cliente ausente no JWT atual; ver `src/middlewares/auth.middleware.ts`.
3. Semântica de assinatura durante a janela de rotação de 24 h.
4. Rate limiting de saída e escala do worker foram adiados.

Essas pendências são parte do design submetido à revisão; não autorizam afirmar que a implementação da feature já foi codificada.
