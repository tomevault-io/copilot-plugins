## traderbot

> Code that trades or backtests lives in `src/` (`strategy.py`, `backtest.py`, `reporting.py`, etc.), while entry points are `run_backtest.py` for historical simulation and `main.py` for live paper trading. Configuration is split between `.env` for Alpaca keys and `config/config.yaml` for strategy knobs. Cached CSVs stay under `data/`, and helper scripts (manual backtest, data fetcher) live in `tools/`. Tests belong in `tests/`, mirroring the module they exercise.

# Repository Guidelines

## Project Structure & Module Organization
Code that trades or backtests lives in `src/` (`strategy.py`, `backtest.py`, `reporting.py`, etc.), while entry points are `run_backtest.py` for historical simulation and `main.py` for live paper trading. Configuration is split between `.env` for Alpaca keys and `config/config.yaml` for strategy knobs. Cached CSVs stay under `data/`, and helper scripts (manual backtest, data fetcher) live in `tools/`. Tests belong in `tests/`, mirroring the module they exercise.

## Build, Test, and Development Commands
Create your environment once: `python3.12 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`. Run a default backtest via `python run_backtest.py`, or point to cached data with `python run_backtest.py --file data/AAPL_historical_2024-01-03_to_2024-01-07.csv`. Live (paper) trading starts with `python main.py` after exporting Alpaca credentials. Use `python -m tools.fetch_historical_data` to refresh cached data and `python -m tools.manual_test_backtest` for quick, CSV-only experiments.

## Coding Style & Naming Conventions
Follow idiomatic Python 3.12: four-space indentation, double quotes for log messages, and snake_case for files, variables, and function names (`Backtester.calculate_current_equity` is the exception because it is a method). Keep modules under 300 lines where practical and favor small, composable classes (see `Backtester`, `Strategy`). Add docstrings for public methods that describe parameters, return types, and side effects. Run `python -m compileall src` if you need a quick syntax sanity check; automated formatters are not configured, so be deliberate.

## Testing Guidelines
Tests live beside their target module under `tests/` (e.g., add `tests/test_backtest.py` when touching `src/backtest.py`). Use `pytest` (`python -m pytest tests -k backtest`) to run focused suites, and seed reproducible data fixtures from sample CSVs in `data/`. Cover both success paths and strategy edge cases (missing columns). Aim to exercise new metrics or trade flows in isolation by mocking Alpaca responses rather than making live calls.

## Commit & Pull Request Guidelines
Recent history (`git log`) shows concise, imperative commit titles such as “Add backtesting arguments” or “Document how to use backtest report stuff.” Match that style, keep the subject under 72 characters, and explain “why” in the body if the diff is non-trivial. Pull requests should include: a summary of behavior changes, reproduction or test instructions (`python run_backtest.py --silent`), any config migrations, and screenshots or metric snippets when altering reports. Reference open issues or plan items in `implementation_plan.md` to tie work to roadmap context.

## Security & Configuration Tips
Never commit `.env`, key files, or raw Alpaca responses—add new secrets to `.gitignore` if needed. Validate configuration changes against `config/config.yaml` defaults and document them in `README.md` or this guide. When working with cached CSVs, strip personally identifiable data before sharing artifacts outside the repository.

---
> Source: [matthewabbott/traderbot](https://github.com/matthewabbott/traderbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
