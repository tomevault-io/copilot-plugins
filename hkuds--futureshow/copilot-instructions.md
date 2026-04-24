## futureshow

> - `futureshow/`: core agent logic (`agent/`, `tool/`, `prompt/`, `utils/`); `main.py` wires configs to agent classes.

# Repository Guidelines

## Project Structure & Module Organization
- `futureshow/`: core agent logic (`agent/`, `tool/`, `prompt/`, `utils/`); `main.py` wires configs to agent classes.
- `configs/default_config.json`: sample run configuration; `.runtime_env.json` is generated to coordinate runtime settings.
- `data/agent_data/<signature>/`: per-model outputs (positions, PnL, trades); `logs/` can store additional runtime logs.
- `frontend/`: static dashboard (`index.html`, `app.js`, `styles.css`, `icons/`) served by `web_server.py`.
- `tests/`: pytest suite covering MCP tools, Polymarket data, and trading flows.
- Helper scripts: `run_agents_once.py` (single pass), `run_agents_loop.py` (repeat rounds), `run_pnl_trackers.py` (PnL logging loop), and `web_server.py` (local API + UI).

## Build, Test, and Development Commands
- Create env & install: `python -m venv .venv && source .venv/bin/activate && pip install -e .[dev]`.
- Run a single round with a config: `python main.py configs/default_config.json` (override via `--config` where supported).
- Sequential agents once: `python run_agents_once.py --config configs/default_config.json --limit 4`.
- Continuous rounds: `python run_agents_loop.py --interval 2400 --overrun-pause 900 --config configs/default_config.json`.
- PnL tracking loop: `python run_pnl_trackers.py --interval 10 --config configs/default_config.json`.
- Serve dashboard/API from repo root: `python web_server.py` then open `http://localhost:8000`.
- Test suite: `pytest -q`.
- Lint/type-check (dev extras): `ruff check futureshow tests` and `mypy futureshow` if types are added.

## Coding Style & Naming Conventions
- Python 3.10+, 4-space indentation, type hints on public functions, meaningful docstrings; keep functions small and pure where possible.
- Use snake_case for Python functions/variables, PascalCase for classes, and descriptive config keys in JSON.
- Frontend is plain JS + DOM: prefer `const`/`let`, camelCase helpers, and keep UI state mutations centralized in `frontend/app.js`.
- Keep logging user-facing text concise and emoji-safe (existing scripts print status with icons).

## Testing Guidelines
- Add tests under `tests/` with `test_*.py` naming; mirror module paths where possible.
- Use pytest parametrization for data-driven cases and fixtures in `tests/conftest.py`.
- For new tools/agents, cover both happy-path and failure handling (API errors, missing config) and assert outputs written to `data/agent_data`.
- Run `pytest` before pushing; add regression samples when fixing bugs.

## Commit & Pull Request Guidelines
- Follow the existing log style: short, imperative/title-case summaries (e.g., `Implement run loop for agents and enhance PnL tracking`); keep to one-line subject.
- PRs should describe intent, configs touched, and expected outputs; link issues when applicable.
- Include screenshots of the dashboard for UI changes and sample command output for new scripts.
- Note any new environment variables or data files created under `data/agent_data`.

## Security & Configuration Tips
- Store API keys (e.g., `OPENAI_API_KEY`, `DEEPSEEK_API_KEY`, `OPENROUTER_API_KEY`, `POLYMARKET_API_KEY`) in a local `.env`; never commit secrets or raw ledger data.
- `RUNTIME_ENV_PATH` defaults to `.runtime_env.json`; scripts write `SIGNATURE` and timestamps there—avoid editing manually.
- Data/ledger files in `data/agent_data` may contain sensitive trades; scrub before sharing artifacts.

---
> Source: [HKUDS/FutureShow](https://github.com/HKUDS/FutureShow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
