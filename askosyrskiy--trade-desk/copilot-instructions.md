## strategies

> Strategy packages emit intents and view models, not broker orders


# Strategies

Implement `trade_desk.strategy.port.Strategy`. Emit `Intent`s (`add`, `close`, `flatten`), not `MultilegOrder`s. Keep rule math in pure functions with tests. Do not import `trade_desk.brokers`. Update `docs/strategies/<id>/` when rules change. Unfrozen interview ambiguities belong in `docs/decisions/`, not in guessed constants.

---
> Source: [askosyrskiy/trade-desk](https://github.com/askosyrskiy/trade-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
