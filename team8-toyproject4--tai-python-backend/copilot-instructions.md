## tai-python-backend

> Locate the FastAPI entry point at `app/main.py`; always instantiate services with `create_app()` so routers and middleware load. Place domain logic inside `app/` (e.g., `app/services/`, `app/repositories/`) and keep request/response schemas in `app/classes/{metadata,requests,responses}.py`. Write tests under `tests/`, mirroring module names such as `tests/api/test_main.py`. Use `dev/` strictly for throwaway notebooks or prototypes and clear outputs before committing. Keep infrastructure files like `Dockerfile` and `docker-compose.yml` at the repository root.

# Repository Guidelines

## Project Structure & Module Organization
Locate the FastAPI entry point at `app/main.py`; always instantiate services with `create_app()` so routers and middleware load. Place domain logic inside `app/` (e.g., `app/services/`, `app/repositories/`) and keep request/response schemas in `app/classes/{metadata,requests,responses}.py`. Write tests under `tests/`, mirroring module names such as `tests/api/test_main.py`. Use `dev/` strictly for throwaway notebooks or prototypes and clear outputs before committing. Keep infrastructure files like `Dockerfile` and `docker-compose.yml` at the repository root.

## Build, Test, and Development Commands
Use `python -m venv .venv && source .venv/bin/activate` to prepare the virtual environment, then install dependencies with `pip install -r requirements.txt`. Run the API locally via `uvicorn app.main:app --reload` for hot reloading. Execute targeted integration checks with `docker-compose up` when external services are required. Validate the suite with `pytest` or narrow focus using `pytest -k name --maxfail=1` to halt on the first failure.

## Coding Style & Naming Conventions
Follow PEP 8 with 4-space indentation and explicit type hints on routes, dependencies, and services. Name modules in snake_case and Pydantic models in PascalCase (`SearchRequest`). Keep route handlers slim by delegating orchestration to helper functions or classes. Respect existing docstring and comment tone; add short explanatory comments only when logic is non-obvious.

## Testing Guidelines
Write tests with `pytest`, leveraging `httpx.AsyncClient` fixtures for API coverage. Confirm each new router is surfaced through `create_app()` by exercising the FastAPI test client. Cover happy paths, validation errors, and regression cases for metadata schemas. Name tests after their behavior (`test_create_search_returns_201`).

## Commit & Pull Request Guidelines
Author imperative, ~50-character commit subjects (`Add search response schema`) and expand with context in the body when needed. Reference issues with `#id`, document user impact, and call out rollback expectations. PRs should summarize functional changes, include verification steps (`uvicorn`, `pytest`), and attach contract evidence when endpoints change.

## Security & Configuration Tips
Load credentials exclusively from `.env` and never hard-code secrets. Scrub notebooks, logs, or sample payloads before committing. Document any new external integrations so operations teams can provision secrets promptly, and ensure docker-compose overrides stay in local files.

---
> Source: [Team8-ToyProject4/tai_python_backend](https://github.com/Team8-ToyProject4/tai_python_backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
