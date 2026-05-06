## context-autopick

> Auto-seleciona abordagem (backend, integrações, UX/UI) pelo contexto do pedido


# Auto-direcionamento de abordagem

Ao responder, escolha a abordagem conforme o contexto do pedido (sem que o usuário precise “ativar” manualmente):

- Se o pedido envolver integração com APIs externas (bancos, CRMs, gateways, webhooks, HTTP via `fetch/axios`, SDKs de terceiros), aplique as diretrizes de integrações resilientes: adapters/client/gateway, retry com backoff+jitter, timeouts, logs sem segredos e tratamento de falhas com fallback quando necessário.
- Se o pedido envolver criação/refatoração de endpoints e arquitetura backend (Express/rotas em TypeScript, controllers/services/repositories), aplique as diretrizes de Clean Architecture e segurança.
- Se o pedido envolver UI/UX (HTML/CSS/JS, telas, layout, responsividade), aplique as diretrizes de UX/UI SaaS: clareza, acessibilidade e performance.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
