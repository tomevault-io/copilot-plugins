## miru

> This document provides essential information for AI agents working on the Miru codebase.

# AGENTS.md - Guidelines for AI Coding Agents

This document provides essential information for AI agents working on the Miru codebase.

## Project Overview

Miru is a personal AI assistant with a **FastAPI backend** (Python) and **React Native frontend** (TypeScript/Expo). It uses Supabase (PostgreSQL + pgvector) as the primary database, and OpenRouter for LLM and embeddings.

**Key Technologies:**
- **Backend:** FastAPI, Supabase Python SDK, CrewAI, PyJWT, Sentry, webauthn
- **Frontend:** React Native, TypeScript, Expo Router, Zustand, NativeWind, @supabase/supabase-js
- **AI:** OpenRouter (LLM + embeddings)
- **Database:** Supabase (PostgreSQL + pgvector)

---

## Build / Test / Lint Commands

### Backend (Python)
```bash
cd backend

.venv/bin/uvicorn app.main:app --reload --host 0.0.0.0 --port 8000  # Run server
.venv/bin/pytest --cov=app --cov-report=term-missing                 # Run all tests
.venv/bin/pytest tests/test_specific.py::test_function_name -v       # Run single test
ruff check . && black --check . && isort --check-only . && mypy app/ # Lint
ruff check --fix . && black . && isort .                             # Auto-fix
```

### Frontend (React Native/TypeScript)
```bash
cd frontend

npx expo start                                        # Run app
npm test                                              # Run all tests
npm test -- -t "test name"                            # Run single test
npm run lint                                          # Lint
npm run type-check                                    # Type check
npm run format                                        # Fix formatting
```

### Make Commands (Root)
```bash
make setup-backend   # Create venv, install deps
make setup-hooks     # Install pre-commit hook
make db              # Start local pgvector Docker container (dev only)
make db-stop         # Stop local Docker container
make test-backend    # Run backend tests
make test-frontend   # Run frontend tests
make test            # Run all tests
make lint-backend    # Lint backend
make lint-frontend   # Lint frontend
make lint            # Lint all
make fix-backend     # Auto-fix backend lint
make fix-frontend    # Auto-fix frontend lint
```

---

## Agent Roster & Responsibilities

The following AI agents actively monitor and modify the Miru codebase. Their actions are automated via `.github/workflows/ai-agents.yml`.

### 1. Jules

**Mission:** Autonomous bug fixing, CodeRabbit review resolution, and Sentry issue remediation.
**Trigger Conditions:**
- Mentioned by the CodeRabbit Bridge in a PR comment (loop limit: 3 rounds).
- Triggered automatically on every issue labeled event and only proceeds when the label equals `jules-fix-pending` (label checked in the job-level filter).
- Scheduled every 6 hours to pull up to 10 open issues labeled `jules-fix-pending`.
- Manual workflow dispatch on an issue labeled `jules-fix-pending`.
- Scheduled weekly (Monday 9 AM UTC) or via manual dispatch to generate a performance report.
**Scope:** Authorized to modify backend Python files, frontend React Native (TypeScript) files, and tests. Not authorized to restructure databases without human approval.
**Note on Prompt:** Jules is instructed to strictly follow project architecture and code style: 100-char lines, type hints/explicit types, double quotes for Python, single quotes for TypeScript, specific import orders (`stdlib` → `third-party` → `first-party`), use `str | None` not `Optional[str]`, use `const` for components/internal functions, include `from __future__ import annotations`, log errors with context (`logger.exception()`), and structure domains in `backend/app/domain/<domain>/` and routes in `backend/app/api/v1/`.

### 2. CodeRabbit

**Mission:** Continuous code review, enforcing style, finding bugs, and suggesting refactors.
**Trigger Conditions:**
- Automatically invoked when the "PR Checks and Linting" CI workflow completes successfully on a PR branch.
- Retries automatically every 30 minutes via a queue processor if rate-limited (label `coderabbit:queued`).
**Scope:** Reviews all modified files in a PR. Posts actionable comments. When 0 actionable comments are found, it triggers the `ai-approved` label.

## Project Structure

```
backend/
  app/
    main.py          # FastAPI entry point, lifespan, CORS, router mounts
    core/
      config.py      # pydantic-settings Settings class, get_settings()
      security/
        auth.py      # Supabase JWT validation, get_current_user() dependency
    domain/          # One package per bounded context
      agents/
        models.py    # Tortoise ORM models
        schemas.py   # Pydantic schemas for agents, capabilities, etc.
        service.py
      auth/
        models.py    # Profile, Passkey Tortoise ORM models
        schemas.py   # Profile, Passkey Pydantic schemas
        service.py
      chat/
        dtos.py      # ChatRoom, ChatMessage Pydantic schemas
        entities.py  # ChatRoom, ChatMessage pure python entities
        service.py
        crew_orchestrator.py # Multi-agent execution
      memory/
        models.py    # Memory, MemoryGraphNode/Edge Tortoise ORM models
        schemas.py   # Memory, MemoryGraphNode/Edge Pydantic schemas
        service.py
        graph_service.py # Graph logic
        document_service.py # Document processing logic
      notifications/
        api/         # Push notification endpoints
        services.py  # Token registration service
      productivity/
        models.py    # Task, Note, CalendarEvent Tortoise ORM models
        schemas.py   # Task, Note, CalendarEvent Pydantic schemas
        entities.py  # Pure python domain entities
        interfaces/  # Interface boundaries for dependencies
        use_cases/   # Application logic orchestrating productivity domain
      agent_tools/
        models.py    # AgentTool, AgentToolLink Tortoise ORM models
        productivity/ # Productivity-specific tool integrations
    infrastructure/
      database/
        base.py      # SupabaseModel — abstract Tortoise base with sql_policies/indexes/functions
        tortoise.py  # TORTOISE_ORM config, init_db(), close_db()
        supabase.py  # Supabase client singleton, get_supabase()
      external/
        openrouter.py  # embed(), stream_chat(), chat_completion() via OpenRouter
        steam.py
        steam_tool.py
      repositories/  # Thin async wrappers around Tortoise queries
    api/
      v1/            # Route handlers, one file per domain area
  tests/
    conftest.py      # Fixtures: TestClient, auth helpers, make_jwt()
    test_agents.py
    test_agents_routes.py
    test_auth.py
    test_health.py
    test_memory.py
    test_openrouter.py
  pyproject.toml     # Dependencies + tool config (ruff, mypy, pytest)
  manage.py          # Migration CLI: makemigrations / check / migrate / status
  .env.example
  Dockerfile

frontend/
  app/                     # Expo Router file-based routing
    (auth)/                # Auth group (login, onboarding)
    (main)/                # Main app group (chat, agents, productivity)
    _layout.tsx            # Root layout with providers
  src/
    components/            # Reusable UI components
    core/
      api/                 # API client (axios), domain services
      models.ts            # TypeScript interfaces/types for domain entities
    features/              # Feature-specific components and logic
    store/                 # Zustand store definitions (useChatStore, useAuthStore, etc.)
  __tests__/               # Jest tests
  package.json
  tsconfig.json
  tailwind.config.js       # NativeWind configuration

supabase/
  migrations/              # Source of truth for schema — committed SQL applied by Supabase CLI
    .schema_snapshot        # DDL snapshot used by manage.py for incremental diffing
    .snapshot_extras        # Serialised sql_policies / sql_indexes from last makemigrations
    .last_checksum          # Content hash used by manage.py check for drift detection
    YYYYMMDDHHMMSS_*.sql    # Timestamped migration files

docker-compose.yml         # Local dev only: pgvector/pgvector:pg16 container
Makefile
```

---

## Architecture

### Backend

Application code is organised into bounded-context packages under `app/domain/`. The key modules are:

- **`app/api/v1/`** — Route handlers, one file per domain area, all mounted under `/api/v1` in `main.py`.
- **`app/domain/**/models.py`** — Tortoise ORM models for each domain (agents, auth, chat, memory, agent_tools).
- **`app/domain/**/schemas.py`** — Pydantic schemas corresponding to the ORM models to maintain clean separation of concerns.
- **`app/domain/**/service.py`** (or **`app/domain/**/use_cases/`**) — Business logic and database operations for each domain.
- **`app/infrastructure/external/openrouter.py`** — All LLM and embedding calls go through here. Models are configurable via env vars (`EMBEDDING_MODEL`, `DEFAULT_CHAT_MODEL`).
- **`app/core/security/auth.py`** — JWT validation (supports HS256 and ES256/RS256 via JWKS). The `get_current_user()` FastAPI dependency is defined here.
- **`app/infrastructure/database/base.py`** — `SupabaseModel`, the abstract Tortoise base that carries `sql_policies`, `sql_indexes`, and `sql_functions` for the migration generator.

### Frontend

The frontend is a **React Native (Expo) application** using **TypeScript**. It follows a structured approach with **Expo Router** for navigation and **Zustand** for state management.

- **`app/` (Expo Router)** — File-based routing. Uses route groups like `(auth)` and `(main)` to separate authentication flows from the main application. Layouts (`_layout.tsx`) handle shared providers and navigation structures.
- **`src/store/` (Zustand)** — Centralised state management. Each domain (auth, chat, agents) has its own store (e.g., `useChatStore`, `useAuthStore`) for managing complex application state and side effects (API calls).
- **`src/core/api/`** — Axios-based API client. Handles authentication headers (Supabase JWT), base URL configuration, and error handling. Domain-specific services use this client to communicate with the FastAPI backend.
- **`src/components/`** — Reusable, atomic UI components styled with **NativeWind** (Tailwind CSS for React Native).
- **`src/core/models.ts`** — Unified TypeScript interfaces for all domain entities, ensuring type safety across the application.

**Authentication:** Handled via `@supabase/supabase-js`. Supports magic links and password-based login. Passkey support is integrated through native modules and backend-coordinated sessions. Sessions are persisted using `expo-secure-store` or `react-native-mmkv`.

### Database

**Supabase (primary):** Accessed via the Supabase Python SDK — not asyncpg directly. All queries use the fluent client API (`.table("x").select(...).eq(...).execute()`). Vector similarity search uses the `match_memories` Supabase RPC function. Row Level Security (RLS) is enabled on all tables. Schema is managed via raw SQL files in `supabase/migrations/` and applied through the Supabase CLI (`supabase db push`).

**Migration system:** Schema changes flow through a custom code-first pipeline:
1. Edit Tortoise ORM models in `app/domain/**/models.py` (including `sql_policies`, `sql_indexes`, `sql_functions` on `Meta`).
2. Run `python manage.py makemigrations <name>` to generate an incremental SQL diff in `supabase/migrations/`.
3. Apply locally with `supabase db reset --local` or push to a remote project with `supabase db push`.

The CI pipeline (`database-migrations.yml`) validates and applies migrations automatically — see the **Migrations** section below.

### API Design
- All routes are under `/api` (e.g., `/api/chat`, `/api/memories`, `/api/agents`)
- Use plural nouns for resources: `/memories`, `/agents`, `/rooms`
- Use nested routes for relationships: `/rooms/{id}/agents`, `/rooms/{id}/messages`
- Auth is enforced via the `get_current_user()` dependency on protected routes

---

## Code Style

### Python

- Line length: 100 characters
- Double quotes for strings, 4-space indentation
- Always include `from __future__ import annotations` at the top
- Use `Black` + `Ruff` for formatting; `isort` (Black profile) for imports
- Import order: stdlib → third-party → first-party (`app.*`)
- Type hints required on all function signatures
- Use `str | None` not `Optional[str]`; `list[str]` not `List[str]`
- Use Google-style docstrings on all public functions and classes
- Log errors with context; use `logger.exception()` for unexpected errors
- Raise `HTTPException` with appropriate status codes in route handlers
- Prefer specific exceptions over bare `except Exception`

**Naming:** `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE` constants, `_leading_underscore` for private

### TypeScript / React Native

- Line length: 100 characters (ESLint/Prettier default)
- Use functional components and React Hooks (no class components)
- `PascalCase` for components, `camelCase` for props, variables, and functions
- Import order: `react` → `react-native` → third-party → internal components → styles/assets
- Prefer interfaces over types for domain entities
- Use absolute imports (e.g., `@/components`, `@/store`)
- Use **NativeWind** (Tailwind CSS) classes for all styling; avoid inline `StyleSheet` objects
- Always use `const` for components and internal functions
- Document complex logic with JSDoc-style comments
- Use optional chaining (`?.`) and nullish coalescing (`??`) appropriately

**Naming:** `ComponentName.tsx` for files, `useStoreName` for Zustand hooks, `domainService.ts` for API layers.

---

## Testing

### Requirements
- Backend: 75%+ coverage; 90%+ on critical paths
- Frontend: 70%+ coverage
- Run tests before committing
- Mock all external services (Supabase, OpenRouter, CrewAI) — never call real APIs in tests
- Write regression tests for every bug fix

### Backend
- Use `pytest` with `pytest-asyncio` (`asyncio_mode = "strict"`) for async tests
- Use fixtures in `conftest.py` for `TestClient`, auth headers, and JWT helpers
- Structure tests as Arrange / Act / Assert
- Use `unittest.mock.patch` to mock Supabase, OpenRouter, and CrewAI
- Test both the happy path and error/edge cases
- Route tests use `fastapi.testclient.TestClient` (synchronous)

### Frontend
- Use **Jest** and **React Testing Library (@testing-library/react-native)**
- Mock all API calls and Zustand stores to isolate component behavior
- Focus on testing user interactions (pressing buttons, entering text)
- Ensure all screens handle loading, error, and empty states gracefully
- Use `screen.getByText`, `screen.getByPlaceholderText` (semantic queries)
- Run tests with `npm test` or `npm test -- --watch` during development

---

## Security

- Never commit secrets — use `.env` (already in `.gitignore`)
- Validate all inputs via Pydantic models and `Field` constraints
- Use parameterized queries via the Supabase SDK — never string interpolation
- Validate Supabase JWTs in `auth.py` middleware; never re-validate in individual routes
- The `SUPABASE_SERVICE_ROLE_KEY` bypasses RLS — use it only in admin-only operations (passkey session minting)
- WebAuthn challenges are stored in-memory with a 5-minute TTL — do not persist them
- Never store tokens in plain text on the frontend (use `expo-secure-store` on frontend)
- Clear auth state on logout via `SupabaseService`

---

## Performance

### Backend
- All I/O must be `async` — never block the event loop
- Paginate all list endpoints; never return unbounded result sets
- Use `hnsw` index on `memories.embedding` for pgvector cosine similarity (already in migrations — do not add a duplicate)
- Stream LLM responses via `stream_chat()` — do not buffer full responses in memory

### Frontend
- Use `const` widgets to prevent unnecessary rebuilds
- Use `FlatList` for long lists — never deeply nested View hierarchies
- Use `useEffect` cleanup returns and proper hook cleanup
- Debounce search inputs
- Use `AsyncStorage` (from `@react-native-async-storage/async-storage`) or `react-native-mmkv` for chat history persistence — do not re-fetch on every render

---

## Error Handling

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | Successful GET / PUT / PATCH |
| 201 | Successful POST (resource created) |
| 204 | Successful DELETE |
| 400 | Validation / bad request |
| 401 | Authentication required |
| 403 | Permission denied |
| 404 | Resource not found |
| 409 | Conflict (e.g., duplicate) |
| 422 | Semantic / business logic error |
| 429 | Rate limit exceeded |
| 500 | Unexpected server error |

### Rules
- Never expose raw exception messages to API consumers
- Always include a machine-readable `error` code alongside the message
- Frontend must never show raw API errors to users — map them to friendly messages
- Provide retry affordance in the UI wherever the operation is safe to retry

---

## Workflow

### Code Editing
- Never create temporary scripts or helper files for one-off tasks
- Always edit the existing codebase directly — no patches, no external tools
- Make incremental, focused changes; avoid batching unrelated modifications
- Follow existing patterns before introducing new ones

### Before Making Changes
1. Read the relevant existing code first
2. Run the tests to confirm the baseline is green
3. Identify any similar implementations to follow

### After Making Changes
1. Run the relevant tests
2. Run linting (`make lint`)
3. Review the diff for unintended side-effects
4. Confirm no secrets ended up in the changeset

---

## Git

### Commit Messages
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `style`
Scopes: `backend`, `frontend`, `api`, `db`, `ui`, `deps`

### Commit Rules
- Small, focused commits — one logical change per commit
- Explain *why*, not *what*, in the commit body
- Reference issue numbers in the footer (`Closes #123`)
- Never commit broken or untested code
- Never commit secrets

### Branch Naming
- `feature/description` — new features
- `bugfix/description` — bug fixes
- `hotfix/description` — urgent production fixes
- `refactor/description` — code cleanup
- `test/description` — test additions

---

## Environment Setup

Copy and populate the backend `.env`:
```bash
cp backend/.env.example backend/.env
```

Required variables:
```
OPENROUTER_API_KEY=your_key_here
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here
SUPABASE_JWT_SECRET=your_jwt_secret_here
WEBAUTHN_RP_ID=localhost
WEBAUTHN_RP_NAME=Miru
WEBAUTHN_EXPECTED_ORIGIN=http://localhost
CORS_ALLOWED_ORIGINS=*

# Optional — defaults are used if unset
# EMBEDDING_MODEL=openai/text-embedding-3-small
# DEFAULT_CHAT_MODEL=anthropic/claude-3.5-sonnet
```

---

## Pre-commit Hook

Runs automatically on `git commit`:
- `ruff check .`
- `black --check .`
- `eslint .`
- `npm run type-check`

Install with: `make setup-hooks`

---

## Common Tasks

### Adding a New API Endpoint
1. Add the route handler in the appropriate `backend/app/api/v1/` file for the domain
2. Add any new Pydantic models to the relevant `app/domain/**/schemas.py`
3. Add business logic to the corresponding `app/domain/**/service.py`
4. Write tests in `backend/tests/`, mocking Supabase and any external services
5. Run `make test-backend` and `make lint-backend`

### Adding a New React Native Screen
1. Create a new route file in `frontend/app/(main)/` (e.g., `new-feature.tsx`)
2. Define the functional component and exports
3. Update navigation if necessary (e.g., adding to bottom tabs in `_layout.tsx`)
4. Create a new store in `src/store/` if the feature manages complex state
5. Add domain-specific API calls to `src/core/api/`
6. Write a component test in `frontend/__tests__/`
7. Run `npm test` and `npm run lint`

### Adding a Database Migration
1. Edit the relevant Tortoise ORM model(s) in `backend/app/domain/**/models.py`.
   - For structural changes (new table, new column): update the model fields.
   - For RLS policies, custom indexes, or trigger functions: update `Meta.sql_policies`,
     `Meta.sql_indexes`, or `Meta.sql_functions` on the model's inner `Meta` class.
2. Generate the incremental migration:
   ```bash
   cd backend
   python manage.py makemigrations <short_description>
   # e.g. python manage.py makemigrations add_user_preferences
   ```
   This diffs the current models against `.schema_snapshot` and writes a new
   `supabase/migrations/YYYYMMDDHHMMSS_<name>.sql` file containing only the delta.
   Use `--full` only when creating the very first migration or intentionally squashing history.
3. Review the generated SQL file before committing — destructive operations (DROP TABLE,
   DROP COLUMN) are emitted as commented-out statements that you must uncomment deliberately.
4. Apply locally to verify:
   ```bash
   supabase db reset --local
   ```
5. Commit the migration file **and** the updated snapshot files
   (`.schema_snapshot`, `.snapshot_extras`, `.last_checksum`) together in the same commit.

The CI pipeline applies migrations automatically via `supabase db push` — see **Migrations** below.

### Working with Vector Embeddings
1. Use `embed()` from `app/openrouter.py` to generate embeddings
2. Store via `store_memory()` in `app/memory.py`
3. Retrieve via `retrieve_memories()` which calls the `match_memories` Supabase RPC
4. The `hnsw` index on `memories.embedding` is already defined in migrations — do not add duplicate indexes

---

## Migrations

### How it works

Schema changes are code-first: you edit Tortoise ORM models, run `manage.py` to generate SQL,
then the Supabase CLI applies that SQL to the database. There is no manual SQL authoring.

```
Edit models.py  →  python manage.py makemigrations <name>  →  supabase db push
```

**`manage.py` commands:**

| Command | Description |
|---------|-------------|
| `python manage.py makemigrations <name>` | Generate an incremental SQL diff from the current models vs. `.schema_snapshot`. Writes `supabase/migrations/YYYYMMDDHHMMSS_<name>.sql` and updates the snapshot files. |
| `python manage.py makemigrations <name> --full` | Generate a complete schema dump (use for the initial migration or to squash history). |
| `python manage.py check` | Exit 1 if models have changed since the last `makemigrations`. Used by CI to catch uncommitted drift. No database required. |
| `python manage.py status` | Show which migration files have / have not been applied (queries `schema_migrations` tracking table). |
| `python manage.py migrate` | Apply pending `.sql` files via `psql`, recording each in a `schema_migrations` tracking table. For local development only — CI uses `supabase db push` instead. |

**Snapshot files** (always commit alongside the migration SQL):

| File | Purpose |
|------|---------|
| `supabase/migrations/.schema_snapshot` | Full DDL snapshot used by `makemigrations` to compute the incremental diff |
| `supabase/migrations/.snapshot_extras` | Serialised `sql_policies` and `sql_indexes` from the last run |
| `supabase/migrations/.last_checksum` | SHA-256 of the last generated schema; used by `manage.py check` |

**Destructive operations** (DROP TABLE, DROP COLUMN) are emitted as commented-out SQL.
You must uncomment them deliberately — they are never auto-applied.

### CI pipeline (`.github/workflows/database-migrations.yml`)

The pipeline runs on every push/PR that touches `supabase/migrations/**` and has three jobs:

**1. `validate-migrations` (all events)**
- Runs `python manage.py check` — fails if any model change was not committed with a migration.
- Starts a local Supabase instance and runs `supabase db reset --local` as a dry-run to verify all SQL files apply cleanly.

**2. `run-migrations-staging` (push to `develop` or manual dispatch)**
- Links to the staging Supabase project via `SUPABASE_PROJECT_REF_STAGING`.
- Runs `supabase db push` to apply any pending migrations.
- Runs `supabase migration list` to verify the applied state.

**3. `run-migrations-production` (push to `main` or manual dispatch)**
- Links to the production Supabase project via `SUPABASE_PROJECT_REF`.
- Runs `supabase db push` to apply any pending migrations.
- Runs `supabase migration list` to verify the applied state.

The migration drift check (`python manage.py check`) also runs as part of `pr-checks.yml`
on every backend PR, so drift is caught at review time before it can reach the migration pipeline.

### Required secrets

| Secret | Used by |
|--------|---------|
| `SUPABASE_ACCESS_TOKEN` | All `supabase` CLI commands |
| `SUPABASE_PROJECT_REF_STAGING` | Staging link step |
| `SUPABASE_PROJECT_REF` | Production link step |
| `SLACK_WEBHOOK_URL` | Failure notification (optional) |

---

## Getting Help

- Check existing code for similar patterns before writing new ones
- Read the tests to understand expected behaviour
- Run `make help` for available commands
- API docs available at `http://localhost:8000/docs` when the backend is running

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YKDBontekoe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YKDBontekoe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
