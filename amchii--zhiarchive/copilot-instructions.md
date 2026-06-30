## zhiarchive

> - `archive/` is the core Python package; `api/` hosts the FastAPI app, `core/` contains monitor/archiver/login workers, `utils/` groups shared helpers, and `config.py`/`env.py` expose settings.

# Repository Guidelines

## Project Structure & Module Organization
- `archive/` is the core Python package; `api/` hosts the FastAPI app, `core/` contains monitor/archiver/login workers, `utils/` groups shared helpers, and `config.py`/`env.py` expose settings.
- Root scripts (`run_api.sh`, `run_archiver.py`, `run_monitor.py`, `run_login_worker.py`, `run_all_workers_in_one.py`) are thin launchers that wire up the package modules.
- `docs/` keeps static screenshots and API UI content, while `states/`, `results/`, `archives/`, and `activities/` house runtime artifacts captured by the workers.

## Build, Test, and Development Commands
- `docker build -t zhi-archive:latest -f CN.Dockerfile .` produces the production-ready image referenced by the compose stacks.
- `docker-compose up -d` spins up all services with the embedded Redis; `docker-compose -f docker-compose2.yaml up -d` lets you point to an external Redis host via `.env`.
- `python -m playwright install chromium` installs browser dependencies before running any worker locally.
- `./run_api.sh` starts the API with the current `.env`/`.apienv`; other `run_*` scripts behave similarly for their workers.
- `python -m pytest` runs the async-aware suite; `ruff check .` enforces the lint rules defined in `pyproject.toml`.

## Coding Style & Naming Conventions
- Stick to 4-space indentation, ≤88 character lines, and the lint rules from `ruff` (no extra formatter unless needed).
- Prefer `snake_case` for functions/modules, `CamelCase` for classes, and `SCREAMING_SNAKE_CASE` for constants; keep exports localized inside `archive/` to avoid circular imports.
- Async helpers end with descriptive verbs (`*_worker`, `*_task`), matching the existing naming in `archive.core`.

## Testing Guidelines
- Follow pytest naming (`test_*.py`/`*_test.py`) so the suite auto-discovers files; the configured `asyncio_default_fixture_loop_scope` keeps async fixtures scoped per function.
- Keep Playwright interactions mocked when possible, document any manual login setup, and store reusable cookies under `states/`.

## Commit & Pull Request Guidelines
- Mirror the repo’s conventional commits (`type(scope): short summary`, e.g., `feat(login): ...`); pick the closest type and scope for new changes.
- PRs should include a clear description, linked issues if available, the commands you ran (build/tests), and any screenshots needed for UI changes.

## Security & Configuration Tips
- Use `.env` and `.apienv` to supply `secret_key` and API credentials; never commit those files or their contents.
- When authentication runs, point the UI to the saved cookie in `states/` or refresh it through the login page, and rotate secrets before sharing logs.

## 使用简体中文与我交流

---
> Source: [amchii/ZhiArchive](https://github.com/amchii/ZhiArchive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
