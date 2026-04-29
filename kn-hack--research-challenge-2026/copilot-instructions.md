## research-challenge-2026

> This project implements a quantitative investment strategy for stock picking using the KaxaNuk

# KN Hack Research Challenge 2026 — Investment Strategy

This project implements a quantitative investment strategy for stock picking using the KaxaNuk
framework. It is organized as three sequential pipelines:

1. **Data Curator** (`run_data_curator.py`) — fetches market and fundamental data, applies custom
   calculations, and outputs enriched datasets to `Data_Curator/`.
2. **Backtest Engine** (`run_backtest_engine.py`) — runs historical simulations against the
   enriched data using `Config/backtest_engine_parameters.xlsx`.
3. **Portfolio Construction** (`run_portfolio_construction.py`) — builds and sizes the final
   portfolio from the signals produced by the Data Curator.

Custom calculation modules live in `Investment_Strategy/src/data_curator/` and are registered in
`run_data_curator.py`. Configuration for the Data Curator is in
`Config/data_curator_parameters.xlsx`.

## Skills
For any code-related task consult the `Investment_Strategy/.agents/skills/code_writing` skill.
For any custom Data Curator calculation task consult the
`Investment_Strategy/.agents/skills/kaxanuk_data_curator_custom_calculation_builder` skill.

## Stack
- Python >=3.12,<3.15
- KaxaNuk Data Curator
- KaxaNuk Backtest Engine

---
> Source: [KN-Hack/Research-Challenge-2026](https://github.com/KN-Hack/Research-Challenge-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
