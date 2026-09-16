## digital-event-manager

> | Path | Language | Package manager | Description |

# AGENTS.md

## Repo layout — three independent projects

| Path | Language | Package manager | Description |
|------|----------|-----------------|-------------|
| `backend/` | Python 3.12 | uv | FastAPI API + PostgreSQL (main deployable) |
| `source/` | Python 3.13 | uv + Makefile | Telegram bot (aiogram) with SQLite |
| `test/` | Java 17 | Maven | Allure-based test suite |

Each project has its own dependencies, lockfile, and `.python-version`. Always run commands from the project directory.

## Backend — commands (all require `uv run` prefix)

```bash
cd backend
uv sync                                  # install deps into .venv
uv run uvicorn app.main:app --reload     # dev server on :8000
uv run alembic upgrade head              # apply migrations (DB must be running)
uv run alembic revision -m "..." --autogenerate  # create new migration
uv run pytest                            # run all tests (mock-based, no DB needed)
uvx ruff check --fix                     # lint
uvx ruff format                          # format
docker compose up --build                # full stack (API + PG 16)
```

## Backend — layered architecture (do NOT break)

```
adapter (FastAPI routers) -> service (business logic) -> repository (SQLAlchemy)
```

Routers import services, never repositories. Schemas are Pydantic models in `app/schemas/`. ORM models in `app/models/`.

## Backend — generated code (critical)

`openapi_server/` is **generated** by `generate_api.sh` / `generate_api.bat` from `resources/api/v1/draw.yaml`. It is **not committed**. CI regenerates it before tests. If you see import errors for `openapi_server.apis.*`, run the generate script first.

The generated routers for draw/draw_results/topics are wired into `app/adapters/api/v1/__init__.py` manually — the class implementations live in `draw.py`, `draw_results.py`, `topics.py` in the same directory.

## Backend — lint quirks

- `alembic/` is excluded from ruff (set in `pyproject.toml`).
- `app/adapters/api/v1/__init__.py` has intentional per-file-ignores `F401, I001` (wildcard imports needed for router wiring).
- Ruff config lives in `backend/pyproject.toml`, not at root.

## Backend — tests

All tests use `MagicMock` / `AsyncMock` with FastAPI `TestClient` + `dependency_overrides`. No real database needed. Located in `tests/`.

## Databases

- **Backend**: PostgreSQL (`digital_events`). Connection URL built in `app/core/config.py` from env vars. Copy `.env.example` to `.env` and set `DB_HOST=localhost` for local dev (default is `db` for Docker Compose).
- **Bot** (`source/`): SQLite, initialized via `make init`.

## Pre-commit

Installed at root: `ruff --fix` + `ruff format` (v0.11.0). Will catch issues before commit.

## Docker notes

The Dockerfile copies `generate_api.sh` and runs it at container start (via `docker-entrypoint.sh`), followed by `alembic upgrade head`, before starting uvicorn. Build includes Java 21 JDK (required by openapi-generator-cli).

---
> Source: [english-it-lab/digital-event-manager](https://github.com/english-it-lab/digital-event-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
