## web4xener

> Cross-species single-cell cell type annotation web app (FastAPI + Vue 3).

# Xener Web — Agent Guidelines

Cross-species single-cell cell type annotation web app (FastAPI + Vue 3).

## Commands

| Where | Command | Purpose |
|-------|---------|---------|
| `frontend/` | `npm run dev` | Vite dev server (port 5173) |
| `frontend/` | `npm run build` | Production build (required before Docker) |
| `frontend/` | `npm test` | Vitest unit tests |
| `frontend/` | `npm run lint` | ESLint with autofix |
| `frontend/` | `npm run test:e2e` | Playwright all browsers |
| root | `python -m backend.main` | Backend dev server (port = `.env` 的 `BACKEND_PORT`,默认 8541;热重载) |
| root | `python -m pytest backend/tests/ -v` | All backend tests |
| root | `python -m pytest backend/tests/ -m unit` | Backend unit tests |

Backend tests need `PYTHONPATH=.`. Integration tests need PostgreSQL + Redis running.

## Architecture

- **Backend entrypoint**: `backend/main.py` — FastAPI app with lifespan that initializes DB/Redis/Xener engine/task queue
- **Frontend entrypoint**: `frontend/src/main.js` — Vue 3 app bootstrap
- **Task runner**: `backend/services/scheduler/task_runner.py` — FIFO subprocess-based, resumes queued tasks on startup
- **Annotation pipeline**: `backend/services/annotation_workflow.py` — 7-step workflow checkpointed via `data_info.yaml`

## Critical Constraints

- **Pinia limited to ReportPage** — `createPinia()` is called in `main.js` solely for `useReportStore` (ReportPage, per Epic 18 / UX-DR1). All other shared state uses module-scoped `ref()` singletons in composables (see `frontend/src/composables/useAuth.js`). New components must NOT use Pinia stores — use composable singletons instead.
- **No bare `import axios`** — ESLint enforces `no-restricted-imports`. Use `apiClient` from `src/api/client.js` (has token interceptor).
- **No Alembic/migrations** — Tables created via `Base.metadata.create_all()` on startup.
- **Config paths must be absolute** — Relative paths cause system exit at startup.
- **Async DB only** — All DB calls use async SQLAlchemy; never use sync DB in routes.
- **Dev mode** (`DEV_MODE=true`) — Skips SMTP, fixes verification code to `000000`, bypasses IP limits.
- **Task ID validation** — No `..`, `/`, `\` allowed in task IDs (path traversal guard).

## E2E Test Gotchas

Read `docs/e2e-test-pitfalls.md` before writing E2E tests. Key points:
- `page.route()` handlers lost after `page.reload()` — re-register
- WebKit excluded from CI (flaky under parallel load)
- Use `Promise.all([waitForResponse, action])` pattern to avoid race conditions
- Firefox/WebKit bfcache may cause stale state on `page.goto('/')` — use `page.reload()`
- Use unique email per test for parallel safety

## Conventions

- Vue files use `@/` alias → `frontend/src/`
- Backend imports: `from backend.module import ...` (absolute)
- Element Plus locale bound in `App.vue` via `<el-config-provider>`, not in i18n singleton (avoids test timeouts)
- All backend config through `backend/core/config.py` — never call `load_dotenv()` in services/routes
- Xener engine is a singleton: `get_xener_engine()` from `backend/core/xener_engine.py`

## Files That Matter

- `design.md` — authoritative business logic spec (Chinese)
- `_bmad-output/project-context.md` — broader project overview
- `docs/e2e-test-pitfalls.md` — required reading for E2E tests
- `docs/ci.md` — CI/CD pipeline and local verification status
- `backend/core/config.py` — all config settings
- `backend/services/annotation_workflow.py` — core annotation pipeline
- `frontend/src/api/client.js` — shared API client with auth interceptor
- `frontend/src/composables/useAuth.js` — canonical pattern for state management without Pinia

## Repo Scope

This is a single-package repo (not monorepo). Frontend dist is served by FastAPI in production — `npm run build` must run before Docker build.

---
> Source: [kipsmith-arch/web4xener](https://github.com/kipsmith-arch/web4xener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
