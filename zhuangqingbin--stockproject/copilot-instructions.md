## stockproject

> This repo is a Python-first monorepo for A-share market tooling. The active app lives under `apps/data_hub/`: `data_pipeline_ts/` for TuShare ingestion and scheduling, `data_pipeline_ak/` for AkShare helpers, and `data_explorer/` for the read-only schema explorer and monitoring UI. Shared runtime helpers live in `shared/stock_core/` (`env.py`, `db.py`, `python_runtime.py`) and `shared/scripts/`. Keep durable docs in `docs/`; treat `experiments/`, `.cache/`, `dist/`, `node_modules/` as generated or exploratory, not canonical source.

# Repository Guidelines

## Project Structure & Module Organization
This repo is a Python-first monorepo for A-share market tooling. The active app lives under `apps/data_hub/`: `data_pipeline_ts/` for TuShare ingestion and scheduling, `data_pipeline_ak/` for AkShare helpers, and `data_explorer/` for the read-only schema explorer and monitoring UI. Shared runtime helpers live in `shared/stock_core/` (`env.py`, `db.py`, `python_runtime.py`) and `shared/scripts/`. Keep durable docs in `docs/`; treat `experiments/`, `.cache/`, `dist/`, `node_modules/` as generated or exploratory, not canonical source.

## Build, Test, and Development Commands
Use Python 3.11+ and prefer the repo resolver: `shared/scripts/resolve_project_python.sh`.

- `python -m pytest -q`: run the Python test suite across configured app test paths.
- `bash apps/data_hub/data_pipeline_ts/scripts/run_daily.sh --help`: inspect the ingestion entrypoint.
- `./apps/data_hub/data_explorer/scripts/run.sh backend`: start the data explorer backend.
- `./apps/data_hub/data_explorer/scripts/run.sh frontend`: start the data explorer frontend (Vite dev mode).
- `npm --prefix apps/data_hub/data_explorer/frontend test`

## Coding Style & Naming Conventions
Follow existing local style instead of introducing a new formatter. Python uses 4-space indentation, type hints, snake_case modules, and `create_app()` factories for FastAPI entrypoints. TypeScript/React uses 2-space indentation, double quotes, PascalCase for components (`CategoryTree.tsx`), and camelCase for helpers/stores (`navigationStore.ts`). Tests use `test_*.py` for Python and `*.test.ts(x)` for frontend units.

## Testing Guidelines
Pytest discovery is defined in `pyproject.toml` and covers `apps/data_hub/tests`, `apps/data_hub/data_pipeline_ts/tests`, `apps/data_hub/data_pipeline_ak/tests`, and `apps/data_hub/data_explorer/tests`. No coverage gate is configured; add focused regression tests for every changed backend module, scheduler path, or UI workflow. Run app-local frontend tests before submitting UI changes.

## Commit & Pull Request Guidelines
Recent history mostly follows Conventional Commit prefixes: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`. Keep that pattern and scope each commit to one app or shared module. PRs should state the affected app(s), list commands run, note any `.env` or schema changes, and include screenshots for frontend updates.

## Security & Configuration Tips
Do not commit secrets. Start from `env.example`; keep real values in `.env` or `.env.local`. Shared MySQL and market-data credentials are consumed through environment variables, so document new keys when adding them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhuangqingbin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zhuangqingbin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
