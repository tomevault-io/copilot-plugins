## strong-stock-screener

> This is an independent strong-stock screener, not the daily-report app.

# AGENTS.md

This is an independent strong-stock screener, not the daily-report app.

- Keep screening status separate from watchlist risk action.
- Never add `empty` as a new-stock screening status.
- `risk_action = "empty"` is only for watchlist or holding risk.
- Keep TickFlow scoped to this screener unless the user explicitly asks to change the daily-report app.
- Prefer small, focused files and tests-first changes.

---
> Source: [icekale/strong-stock-screener](https://github.com/icekale/strong-stock-screener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
