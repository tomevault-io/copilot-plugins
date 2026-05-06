## ucp-n8n-workflows-json

> n8n workflows exportados (JSON) - idempotência e segurança


# N8n Workflows JSON UCP

Ao editar workflows exportados em JSON:

- Valide triggers: garanta que o workflow seja tolerante a reprocessamentos (idempotência).
- Dedupe eventos quando houver reenfileiramento/retry do provedor (ex.: `event_id`).
- Não logue payloads com PII/segredos; revise campos que vão para `data`, `notes` ou `debug`.
- Garanta que chamadas externas tenham retry controlado (quando fizer sentido) e timeouts.
- Responda rápido quando o workflow for acionado via webhook (use 202 quando possível e processe o resto async).

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
