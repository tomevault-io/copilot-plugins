## quant-factory

> QuantFactory is a single-user A-share research platform for AI-generated,

# Repository Guidelines

## Project Purpose & Decision Policy

QuantFactory is a single-user A-share research platform for AI-generated,
self-optimizing strategies and faithful daily-market simulation. Backtest and simulated
trading accuracy outrank feature breadth or UI speed. Live brokerage integration and
multi-user features are out of scope; a Web UI is required but may come last.

Ask the repository owner before implementing uncertain requirements, assumptions,
architecture choices, or thresholds. Never invent defaults that affect results,
risk, scope, or validation conclusions.

## Project Structure & Module Organization

`src/` contains data, backtest, strategy, optimization, anti-overfit, evaluation, and
protocol packages. Keep `cli/` and Web routes thin; reusable research logic belongs
in `src/`. Tests in `tests/` mirror features (`test_backtest_runner.py`). Architecture
and gates live in `docs/`; protocol templates live in `data/protocols/`.

## Build, Test, and Development Commands

- `conda env create -f environment.yml` creates the Python 3.11 environment.
- `conda run -n quant_factory python -m pip install -e ".[dev]"` installs the
  package and contributor tools in editable mode.
- `conda run -n quant_factory python -B -m pytest -q` runs the full regression suite.
- `conda run -n quant_factory ruff check src cli tests` performs static linting.
- `conda run -n quant_factory black --check src cli tests` verifies formatting.

## Coding Style & Naming Conventions

Use four-space indentation, Python 3.11 syntax, type hints, and concise docstrings for
public behavior. Black and Ruff use a 100-character line length. Use `snake_case` for
modules/functions, `PascalCase` for classes, and `UPPER_SNAKE_CASE` for constants.

## Testing Guidelines

Pytest discovers `tests/test_*.py`; name tests `test_<behavior>`. Add regressions and
boundary tests for dates, A-share rules, leakage, accounting, and fail-closed paths.
Mark local-data tests `integration`. No numeric coverage target is enforced; scenario
quality is the gate. Compare representative strategies with JoinQuant or another
platform using identical dates, prices, costs, timing, and universe. Explain every
discrepancy; never tune assumptions just to force a match.

## Commit & Pull Request Guidelines

History uses phase-oriented subjects such as `P5 ...` and `M0 ...`. Prefer
`P#/M#: imperative summary` or `scope: imperative summary`. Pull requests must state
rationale, affected phase/protocol, tests, and limitations; include screenshots for
Web or HTML report changes.

## Security & Research Constraints

Never commit `.env`, tokens, private keys, raw data, Qlib stores, reports, or logs.
Treat `docs/PLAN.md` as the source for research gates. Existing P5 output is
`EXPERIMENTAL` and cannot be promoted. Preserve the Tushare 2100-point limit and never
expose locked OOS data to generation or optimization paths.

---
> Source: [ximilu0114-droid/quant-factory](https://github.com/ximilu0114-droid/quant-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
