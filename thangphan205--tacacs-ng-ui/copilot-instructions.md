## tacacs-ng-ui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**tacacs-ng-ui** is a full-stack web application providing a graphical interface for managing [tac_plus-ng](https://github.com/MarcJHuber/event-driven-servers) TACACS+ server configurations. It generates and validates TACACS+ config files, manages users/groups/policies, and displays AAA accounting statistics.

## Commands

### Backend (run from `backend/`)

```bash
uv sync                          # install/sync Python dependencies
fastapi dev app/main.py          # local dev server with hot reload
uv run pytest                    # run all tests
uv run pytest tests/api/routes/test_hosts.py  # run a single test file
uv run pytest -k "test_name"     # run a single test by name
uv run bash scripts/lint.sh      # ruff lint
uv run bash scripts/format.sh    # ruff format + fix
uv run bash scripts/prestart.sh  # run DB migrations + seed initial data
```

### Frontend (run from `frontend/`)

```bash
npm run dev              # Vite dev server (port 5173)
npm run build            # TypeScript check + production build
npm run lint             # Biome format + lint (writes in place)
npm run generate-client  # regenerate TypeScript API client from OpenAPI schema
```

### Docker Compose (run from root)

```bash
docker compose watch     # full dev stack with hot reload (recommended)
docker compose up -d     # start all services detached
docker compose logs -f backend  # tail backend logs
docker compose exec backend bash  # shell into backend container
```

### API Client Generation

After changing any backend route or model:

```bash
bash scripts/generate-client.sh  # from root — exports OpenAPI schema and regenerates frontend/src/client/
```

### Database Migrations (inside backend container or local venv)

```bash
alembic revision --autogenerate -m "description"  # generate migration from model changes
alembic upgrade head                               # apply pending migrations
```

### Pre-commit

```bash
uv run pre-commit install        # install hooks
uv run pre-commit run --all-files
```

## Architecture

### Backend (`backend/app/`)

Follows a strict three-layer pattern:

- **`models.py`** — All SQLModel schemas in one file (SQLModel convention). Every entity has at least a DB model and a `*Public` DTO. Base class `TimestampModel` adds UTC `created_at`/`updated_at`.
- **`crud/`** — Business logic only. Must **not** raise `HTTPException`. Returns model instances or `None`. TACACS-specific CRUD modules (e.g. `tacacs_configs.py`) write config files to `/app/tacacs_config/`, call external validators, and generate MAVIS/LDAP templates.
- **`api/routes/`** — HTTP layer only. Translates CRUD results to HTTP responses/status codes. Uses shared deps from `api/deps.py`: `SessionDep`, `CurrentUser`, `get_current_active_superuser`.

Auth is JWT (HS256, 8-day expiry) via `core/security.py`. Settings (env vars) live in `core/config.py` as a Pydantic `Settings` class.

### Frontend (`frontend/src/`)

- **`client/`** — Auto-generated from backend OpenAPI spec. **Never edit manually.** Regenerate with `npm run generate-client`.
- **`routes/`** — TanStack Router file-based routing. `_layout.tsx` wraps all authenticated routes. Route tree is auto-generated into `routeTree.gen.ts`.
- **`components/`** — Chakra UI v3 components. Prefer generic/reusable components over per-entity duplicates.
- **`main.tsx`** — Configures OpenAPI client base URL (`VITE_API_URL` env var, set at build time) and JWT token from `localStorage.access_token`. Auto-logouts on 401/403.

Server state is managed with TanStack React Query v5. Dark mode via next-themes.

### Infrastructure

Docker Compose services: `db` (PostgreSQL 18), `backend` (FastAPI), `frontend` (Vite/Nginx), `traefik` (reverse proxy), `adminer` (DB UI), `mailcatcher` (SMTP testing).

Local dev URLs:
- Frontend: http://localhost:5173
- Backend / Swagger: http://localhost:8000/docs
- Adminer: http://localhost:8080
- Traefik: http://localhost:8090
- MailCatcher: http://localhost:1080

### TACACS+ Config Generation

The core domain logic is in `backend/app/crud/tacacs_configs.py`. It reads/writes config files to `/app/tacacs_config/` (Docker volume), validates syntax using the `tac_plus-ng` binary, and generates MAVIS LDAP configuration blocks from template strings. Config changes go through this layer before being applied.

## Code Conventions

### Python

- Python 3.12+ union syntax: `str | None`, `list[str]` — never `Optional`, `List`
- Always timezone-aware datetimes: `datetime.now(timezone.utc)`, never `datetime.utcnow()`
- `logging` module only, never `print()` in production code
- Return type annotations required on all functions
- Error messages in `HTTPException` must match the actual entity (avoid copy-paste errors)

### TypeScript / React

- No `any` — use proper types or `unknown`
- Hooks only inside React components or custom hooks, never in plain functions
- Indentation: 2 spaces, double quotes, Biome enforced

### API Patterns

- Pagination: `skip`, `limit`, `sort_by`, `sort_order` query params
- UUIDs as primary keys throughout
- `operationId` format `{tag}-{name}` (controls generated client method names)

---
> Source: [thangphan205/tacacs-ng-ui](https://github.com/thangphan205/tacacs-ng-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
