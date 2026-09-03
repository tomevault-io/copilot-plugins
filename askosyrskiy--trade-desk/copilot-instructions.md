## trade-desk

> trade-desk architecture and safety


# trade-desk

Specs in `docs/specs/` are source of truth. Change the spec before changing ports.

- Engine talks only to `Broker` and `Strategy` ports. Do not import Tradier from a strategy, or iron-fly types from a broker adapter.
- Iron fly is `confirm` mode: compute Watch / Next Move / BTC; do not place until the UI confirms.
- Never commit `.env`, tokens, account IDs, or live fills.
- New broker = `src/trade_desk/brokers/<name>/` implementing `Broker`. New strategy = `src/trade_desk/strategies/<name>/` plus `docs/strategies/<id>/`.
- 4-leg live orders use a credit/debit limit, not market.

---
> Source: [askosyrskiy/trade-desk](https://github.com/askosyrskiy/trade-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
