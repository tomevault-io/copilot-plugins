## byaan

> - **Backend format/lint:** `cd server && uv run ruff format . && uv run ruff check --fix .`

# AGENTS.md - Agent Coding Guidelines

## Build/Test Commands

- **Backend format/lint:** `cd server && uv run ruff format . && uv run ruff check --fix .`
- **Backend tests:** `cd server && PYTHONPATH=..:tests uv run pytest`
- **Single test:** `cd server && PYTHONPATH=..:tests uv run pytest tests/test_file.py::test_function`
- **Frontend build/lint:** `cd client && pnpm build` / `cd client && pnpm lint`
- **Tauri dev:** `cd client && pnpm tauri:dev`
- **Docker:** `make dev` (start) / `make stop` (stop)

## Code Style

**Python:** Ruff (line-length=120, py311+), double quotes, snake_case functions/vars, PascalCase classes

- Imports: sorted alphabetically; first-party (`server`, `models`, `routers`, etc.) after third-party
- Async: use `asyncio` with `pytest-asyncio` (asyncio_mode=auto)
- Errors: return `error_response()` from `schemas.standard_response`; use custom logger: `from server.utils.custom_logger import get_logger`
- Type hints required (Pydantic models, async generators)

**TypeScript/React:** ESLint configured, functional components + hooks, run `pnpm lint` before commit

## Architecture

- `server/` - FastAPI backend (routers/, services/, models/, schemas/, repositories/, tests/)
- `client/` - React + Tauri (src-tauri/)
- SQLAlchemy 2.0+ async, Alembic migrations, SQLite/PostgreSQL/MongoDB/DuckDB support
- Test markers: `@pytest.mark.notebook`, `.connection`, `.workflow`, `.error_handling`

---
> Source: [byaan-ai/byaan](https://github.com/byaan-ai/byaan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
