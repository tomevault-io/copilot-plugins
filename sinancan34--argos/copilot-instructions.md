## argos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Argos is a Pixel Code Audit Tool — a Chrome extension that automates browser actions and validates tracking pixel fires against user-defined rules. It has a FastAPI backend for scenario persistence and a Chrome extension frontend.

- **Backend:** FastAPI + SQLAlchemy + Alembic, Python 3.12
- **Extension:** WXT (Manifest V3) + React 19 + TypeScript + Tailwind CSS v4 + shadcn/ui

### Prerequisites

- Python 3.12+
- Node.js 18+
- Google Chrome

## Commands

### Backend

All commands run from `backend/` with the virtual environment activated (`source env/bin/activate`).

```bash
pip install -r requirements.txt        # Install dependencies
python -m uvicorn app.main:app --reload # Run dev server (http://127.0.0.1:8000)
alembic upgrade head                    # Run migrations
alembic revision --autogenerate -m "description"  # Generate migration
alembic downgrade -1                    # Rollback one migration
pytest                                  # Run all tests
pytest tests/test_scenarios_api.py      # Run a single test file
pytest tests/test_scenarios_api.py::test_create_scenario -v  # Run a single test
pytest --cov=app --cov-report=term-missing  # Run tests with coverage
```

Copy `.env.example` to `.env` before first run. Tests use an in-memory SQLite database (see `tests/conftest.py` for fixtures). No linter or formatter is configured.

### Extension

All commands run from `extension/`.

```bash
npm install          # Install dependencies
npm run dev          # Dev mode (opens Chrome with extension loaded)
npm run build        # Production build → extension/.output/chrome-mv3/
npm run test         # Run all tests (vitest)
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
```

No lint or typecheck scripts are configured. Tests use Vitest.

### Running Together

Both the backend and extension must run simultaneously. Start the backend dev server first, then `npm run dev` in the extension directory. The extension connects to `http://127.0.0.1:8000` by default (configurable via `VITE_API_BASE_URL` at build time).

## Shared Definitions

The `shared/` directory contains JSON files that are the single source of truth for both backend and extension:

- `shared/commands.json` — Command definitions (goto, click), selector strategies, match types
- `shared/validations.json` — Field constraints for scenarios, parameters, and selectors; enum values for sortBy, sortOrder
- `shared/providers.json` — Tracking provider definitions (e.g., Google Analytics) with URL patterns and parameter suggestions for autocomplete

Both sides load from these files: backend via `command_registry.py` / `validation_registry.py`, extension via `lib/commands.ts` / `lib/validation-registry.ts`. When adding or modifying commands, parameters, or validation rules, update the shared JSON first — both sides derive from it.

**Validation registry pattern:** `validations.json` defines field constraints (`type`, `required`, `minLength`, `maxLength`, `min`, `max`, `positive`, `minItems`, `conditionalRequired`). The backend converts these to Pydantic `Field()` kwargs via `pydantic_field_kwargs()`. The extension converts them to Zod schemas via `buildStringSchema()`, `buildIntSchema()`, `buildEnumSchema()`. Enum fields reference other JSON sources (e.g., `"source": "commands.matchTypes"`). Conditional validation (e.g., `value` required only when `match` is not `exists`) uses `@model_validator(mode="after")` in Pydantic and `.refine()` in Zod.

## Backend Architecture

**Domain model:** `Scenario` → has many `TestRun` → has many `TestRunResult`. All PKs are UUID strings generated in Python. Timestamps are ISO 8601 strings (not SQL datetime), generated via `datetime.now(timezone.utc).isoformat()`. JSON columns store `steps`, `validations` (on Scenario) and `actual_value` (on TestRunResult).

**Response envelope:** Single resource: `{ data: {...} }`. List resource: `{ data: [...], meta: { page, size, total_count, total_pages }, links: { self, first, last, next, prev } }`. Delete resource: `{ data: { id: "..." } }`. Error: `{ error: { code, message, details[] } }`.

**API versioning:** All routes are under `/api/v1/` prefix. Health check is at `/health` (root level).

**Key patterns:**
- Database sessions via FastAPI dependency injection: `db: Session = Depends(get_db)`
- All ORM models inherit from `Base` (in `database.py`) and live in `app/models.py`
- Pydantic schemas (request/response) and enums (MatchType, SelectorStrategy, SortBy, SortOrder) live in `app/schemas.py`, separate from ORM models
- New models must be imported in `alembic/env.py` for autogenerate to detect them
- Business logic lives directly in routers (no services layer)
- Config uses Pydantic BaseSettings with `.env` file loading
- SQLite by default (`sqlite:///./argos.db`), configurable via `DATABASE_URL`
- `connect_args={"check_same_thread": False}` is set for SQLite compatibility — must be adjusted if switching to PostgreSQL
- CORS origins configurable via `CORS_ALLOWED_ORIGINS` env var (comma-separated, defaults to `*`)
- Rate limiting via `slowapi` — configurable via `RATE_LIMIT` env var (default: `60/minute`)
- Custom exception handlers produce consistent error envelope: `{ error: { code, message, details[] } }`
- Raise `AppException` (from `app/exceptions.py`) in routers for structured errors — it carries `status_code`, `code`, `message`, and `details[]`
- `status` field is a string enum (`"active"` / `"inactive"`), not an integer

## Extension Architecture

**Key patterns:**
- HTTP client: `ky` instance in `lib/api/client.ts` with `prefixUrl` set to `${VITE_API_BASE_URL}/api/v1`. A `beforeError` hook auto-parses backend error envelopes via `parseApiError()` (in `lib/api/errors.ts`) and attaches structured error details to the error object. API functions (e.g., `lib/api/scenarios.ts`) use this client and return typed response envelopes.
- Server state via Tanstack Query (`retry: 1`, `staleTime: 30s`), forms via React Hook Form + Zod
- Hash-based routing via Tanstack Router (required for extension side panel)
- `@/` path alias maps to `src/`
- Zod v4 schemas in `lib/schemas/` mirror backend Pydantic schemas — keep them in sync when changing either side
- Drag-and-drop step reordering via `@dnd-kit`; collapsible steps via Radix primitives
- Chrome permissions (configured in `wxt.config.ts`): `sidePanel`, `activeTab`, `tabs`, `scripting`, `webNavigation`, `webRequest` + host permission `<all_urls>`

## Step Execution Architecture

The extension executes scenario steps via a three-layer messaging system:

```
Side Panel ──port──▶ Background (orchestrator) ──one-shot──▶ Content Script
            ◀──port──                            ◀──response──
```

**Flow:**
1. Side panel connects via `chrome.runtime.connect()` on port `"argos-execution"`
2. Sends `EXECUTE_STEPS` or `EXECUTE_SCENARIO` with steps array and timeouts
3. Background orchestrator creates a new tab, runs steps sequentially
4. For each step: emits `STEP_START` → executes command → emits `STEP_SUCCESS` or `STEP_ERROR`
5. `goto` command: navigates tab + injects content script + waits for `CONTENT_READY`
6. `click` command: sends `EXEC_CLICK` one-shot message to content script with selector entries
7. Content script finds elements via CSS/XPath/linkText strategies and performs the action
8. Final `EXECUTION_COMPLETE` message carries all step results

**Cancellation:** Execution aborts if the port disconnects (side panel closed) or the execution tab is closed. A concurrency guard (`let executing = false`) prevents overlapping runs.

## Network Capture & Validation

After step execution, the orchestrator captures network requests and validates them against scenario-defined rules:

1. `network-capture.ts` — Attaches `chrome.webRequest.onBeforeRequest` listener to the execution tab, collects all outgoing URLs **and request bodies** (decodes `raw` bytes and `formData` from `WebRequestBody`) during a configurable timeout window
2. `validator.ts` — Evaluates each validation rule against captured requests: matches URL against provider patterns, then builds param sets from both URL query strings and request body lines, then checks parameter matches against those sets. A validation passes if **any** captured request satisfies all its parameter rules.
3. `lib/executor/matchers.ts` — Pure matching functions (`checkUrlMatch`, `checkParamMatch`, `parseQueryParams`, `parseBodyLines`, `buildParamSets`) supporting match types: `exists`, `exact`, `contains`, `startsWith`, `endsWith`, `regex`

**Request body handling:** Body lines are split on `\r?\n`, each line is URL-decoded and parsed as query params. `buildParamSets()` produces one param set per body line (merged with URL params), so a validation passes if any single event/line in the payload satisfies all rules.

**GA4 item parameter parsing:** `parseGA4Items()` expands GA4's compact `pr1`, `pr2`, ... item tokens. Each token is `~`-delimited with known prefixes (`id`, `nm`, `ln`, `va`, `qt`, `lp`, `pr`, `br`, `ca`, `c2`–`c5`, `ds`) plus custom `k#`/`v#` pairs. Parsed items are accessible via dot notation keys like `pr1.nm`, `pr1.id`, enabling validation rules such as `{ key: "pr1.nm", match: "exact", value: "barista" }`.

**Two execution modes:** `step-test` (runs steps only, no validations) and `scenario-run` (runs steps + captures network + evaluates validations). The mode determines whether `validationResults` appears in the `EXECUTION_COMPLETE` message.

## Commit Convention

Conventional commits: `feat:`, `chore:`, `docs:`, `fix:`, etc.

## Additional Directives

See `AGENT_DIRECTIVES.md` for hard override rules governing pre-work protocol, code quality, context management, edit safety, and commit discipline. See `CLAUDE_MODIFY.md` for project-specific extensions to those directives.

---
> Source: [sinancan34/argos](https://github.com/sinancan34/argos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
