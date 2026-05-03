## ibkr-scanner

> - `src/` holds the scanner code: `main.py` (entrypoint), `app.py` (IBKR client wrapper), `aggregation.py` (5s → 1m bars), `strategy.py` (signal logic), `alerts.py` (Telegram), `config.py` (env-driven settings).

# Repository Guidelines

## Project Structure & Module Organization
- `src/` holds the scanner code: `main.py` (entrypoint), `app.py` (IBKR client wrapper), `aggregation.py` (5s → 1m bars), `strategy.py` (signal logic), `alerts.py` (Telegram), `config.py` (env-driven settings).
- `logs/` contains runtime logs (default `logs/scanner.log`).
- `.env.example` is the configuration template; copy to `.env` for local runs.
- `Dockerfile` and `docker-compose.yml` define the container runtime.

## Build, Test, and Development Commands
- Create a venv and install dependencies:
  - `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
- Run locally:
  - `cp .env.example .env`
  - `python -m src.main`
- Run in Docker:
  - `docker compose up --build`

## Coding Style & Naming Conventions
- Python 3.11+; use 4-space indentation and PEP 8-style naming (`snake_case` for functions/vars, `CapWords` for classes).
- Keep configuration in `src/config.py` and `.env` rather than hard-coding.
- Log operational events; prefer structured, readable messages since logs are a primary debug path.

## Testing Guidelines
- No test suite is included in this repo.
- If adding tests, place them in a new `tests/` folder and name files `test_*.py` to match common pytest discovery.
- Document how to run tests (e.g., `pytest`) if you introduce a framework.

## Commit & Pull Request Guidelines
- This checkout does not include Git history, so no established commit convention is visible.
- Use concise, imperative commit subjects (e.g., “Add VWAP filter to strategy”).
- PRs should describe behavior changes, include reproduction steps, and note any config/env changes (especially `.env` keys).

## Configuration & Operations Notes
- IBKR connectivity requires TWS or IB Gateway with API access enabled; market data subscriptions must match the scan universe.
- Telegram alerts require a bot token and chat ID in `.env`.
- Default scanner settings are conservative; adjust via `.env` to tune scan size and subscriptions.

---
> Source: [3ltrashpanda/ibkr_scanner](https://github.com/3ltrashpanda/ibkr_scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
