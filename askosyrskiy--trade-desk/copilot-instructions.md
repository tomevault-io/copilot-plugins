## broker-adapters

> Broker adapters map venue HTTP to domain types only


# Broker adapters

Implement `trade_desk.broker.port.Broker`. Return `domain` models only. Copy `occ_symbol` from the chain; do not construct OCC strings by hand. Preview before place. Poll order status — a 200 on submit is not a fill. No strategy imports in this tree.

---
> Source: [askosyrskiy/trade-desk](https://github.com/askosyrskiy/trade-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
