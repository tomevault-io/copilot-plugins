## librecodeinterpreter

> Core application code lives in `src/`. Use `src/api/` for FastAPI routes, `src/services/` for orchestration and business logic, `src/services/sandbox/` and `src/services/container/` for execution backends, `src/models/` for request/response models, and `src/config/` for environment-driven settings. Supporting docs are in `docs/`, dashboard assets in `dashboard/`, container/runtime files in `docker/`, and helper scripts in `scripts/`.

# Repository Guidelines

## Project Structure & Module Organization
Core application code lives in `src/`. Use `src/api/` for FastAPI routes, `src/services/` for orchestration and business logic, `src/services/sandbox/` and `src/services/container/` for execution backends, `src/models/` for request/response models, and `src/config/` for environment-driven settings. Supporting docs are in `docs/`, dashboard assets in `dashboard/`, container/runtime files in `docker/`, and helper scripts in `scripts/`.

Tests are split by scope: `tests/unit/` for isolated service logic, `tests/integration/` for API and dependency-backed flows, `tests/functional/` for live endpoint testing, and `tests/snapshots/` for stored response fixtures.

## Build, Test, and Development Commands
Set up a local environment with:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Run locally with `uvicorn src.main:app --reload`. Start required services with `docker compose up -d`, and build the sandbox image with `docker build -t code-interpreter:nsjail .`.

Key verification commands:

```bash
pytest tests/unit/
pytest tests/integration/
pytest tests/functional/ -v
pytest --cov=src tests/
black src/ --check
flake8 src/
mypy src/
bandit -r src/ -s B104,B108 --severity-level high
```

## Coding Style & Naming Conventions
Target Python 3.11+ with 4-space indentation, explicit type hints, and small async-friendly service boundaries. Follow Black formatting and keep code Flake8- and MyPy-clean. Use `snake_case` for modules, functions, and variables; `PascalCase` for classes and Pydantic models; and `UPPER_SNAKE_CASE` for constants and env names.

## Testing Guidelines
Pytest, `pytest-asyncio`, and `pytest-cov` are the standard tools. Name files `test_*.py` and mirror the component under test where practical, for example `tests/unit/test_session_service.py`. Add unit coverage for new logic first, then integration coverage for endpoint or storage changes. Functional tests use `API_BASE`, `API_KEY`, and `API_TIMEOUT`; keep them stable against a real running API.

## Commit & Pull Request Guidelines
Recent history uses short imperative subjects with prefixes such as `fix:`, `docs:`, `chore(...)`, and `feat:`. Keep the first line under 72 characters and reference issues in the body when relevant. Pull requests should explain behavior changes, note config or API contract impacts, and include the commands you ran. Add screenshots when changing the admin dashboard or other visible UI.

## Security & Configuration Tips
Never commit populated `.env` files, API keys, or storage credentials. Use `.env.example` as the template, and review `docs/CONFIGURATION.md` and `docs/SECURITY.md` before changing auth, sandboxing, Redis, or MinIO behavior.

---
> Source: [usnavy13/LibreCodeInterpreter](https://github.com/usnavy13/LibreCodeInterpreter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
