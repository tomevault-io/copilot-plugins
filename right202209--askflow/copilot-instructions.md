## askflow

> Core backend code lives in `src/askflow/`, split by domain (`agent/`, `rag/`, `chat/`, `ticket/`, `admin/`) with shared infrastructure in `core/`, ORM models in `models/`, and Pydantic schemas in `schemas/`. Database migrations live in `alembic/versions/`. Utility scripts such as `scripts/seed_data.py` and `scripts/create_user.py` handle local setup tasks. Tests are organized under `tests/unit`, `tests/integration`, and `tests/e2e`. The frontend lives in `web/src/` with `components/`, `pages/`, `services/`, `stores/`, and `types/`. Treat `web/dist` and `static/dist` as build output, not source.

# Repository Guidelines

## Project Structure & Module Organization
Core backend code lives in `src/askflow/`, split by domain (`agent/`, `rag/`, `chat/`, `ticket/`, `admin/`) with shared infrastructure in `core/`, ORM models in `models/`, and Pydantic schemas in `schemas/`. Database migrations live in `alembic/versions/`. Utility scripts such as `scripts/seed_data.py` and `scripts/create_user.py` handle local setup tasks. Tests are organized under `tests/unit`, `tests/integration`, and `tests/e2e`. The frontend lives in `web/src/` with `components/`, `pages/`, `services/`, `stores/`, and `types/`. Treat `web/dist` and `static/dist` as build output, not source.

## Build, Test, and Development Commands
Use `make install` to install backend dependencies and `make install-web` for the frontend. Start local infrastructure with `make docker-up`, then apply schema changes with `make migrate`. Run the API with `make dev` and the Vite app with `make dev-web`. Backend tests run with `make test`. Python lint/format checks use `make lint`, and `make format` applies Ruff formatting. Frontend production validation is `cd web && npm run build`.

## Coding Style & Naming Conventions
Target Python 3.11+, 4-space indentation, explicit type hints, and small async-friendly services. Follow existing backend naming: `snake_case` modules and functions, `PascalCase` classes, and domain-focused routers/services/repos. Ruff enforces Python formatting with a 100-character line length. In `web/src`, keep React components and page files in `PascalCase` (for example, `ChatPage.tsx`), and use `camelCase` for hooks, stores, and service modules (for example, `useWebSocket.ts`, `authStore.ts`). Preserve the current TypeScript style: strict typing, `@/` imports, double quotes, and semicolons.

## Testing Guidelines
Backend tests use `pytest`, `pytest-asyncio`, and `pytest-cov`; `pyproject.toml` already enables coverage for `src/askflow`. Name files `test_*.py` and keep unit tests close to the behavior being validated. Add integration tests for database, Redis, or API boundary changes, and e2e tests for full workflows. There is no dedicated frontend test runner yet, so at minimum run `npm run build` after UI changes.

## Commit & Pull Request Guidelines
Recent history follows Conventional Commit prefixes such as `feat:` and `fix:`; keep using that pattern with short, imperative summaries. PRs should describe the user-visible change, call out schema or `.env` updates, list commands run (`make test`, `make lint`, `npm run build`), and include screenshots for frontend changes. Never commit secrets from `.env`; use `.env.example` as the template for new settings.

---
> Source: [Right202209/AskFlow](https://github.com/Right202209/AskFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
