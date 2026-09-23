# ADR-004 — HMAC por endpoint e rotação de secret

**Evidências do [inventário](../EVIDENCIAS.md):** T-19, T-20, T-21, T-22.

## Status
Decidido na reunião; aguardando revisão do RFC.

## Contexto
Pedidos serão enviados para fora da infraestrutura, exigindo autenticação de origem e integridade do corpo. Uma secret global ampliaria o impacto de vazamento. [09:19–09:21] Sofia.

## Decisão
Assinar o corpo enviado com HMAC-SHA256 e publicar a assinatura em `X-Signature`. Cada endpoint possui secret própria. A API permite rotação; a secret anterior continua válida por 24 horas e então expira. Exigir HTTPS e recusar payload acima de 64 KB. [09:20–09:24] Sofia, Diego e Larissa.

## Alternativas consideradas
- Secret global: um vazamento comprometeria todos os endpoints. [09:21] Sofia.
- HTTP sem TLS: explicitamente recusado na validação da URL. [09:23] Sofia.

## Consequências
- Positiva: comprometimento de uma secret fica limitado ao endpoint e a rotação permite migração.
- Negativa: armazenamento e distribuição seguros da secret e convivência temporária de duas versões exigem cuidado; a assinatura deve usar os bytes exatos do corpo enviado.
