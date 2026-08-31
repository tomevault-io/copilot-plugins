## spring-batch-dashboard

> Top-level components. Each has its own `AGENTS.md` with stack/layout/conventions specific to it. Use this file for the cross-cutting picture.

# Repo agent guide

Top-level components. Each has its own `AGENTS.md` with stack/layout/conventions specific to it. Use this file for the cross-cutting picture.

```
spring-batch-dashboard/
├── backend/        Spring Boot 4 / Java 21 REST API (reads BATCH_* metadata)
│   AGENTS.md       — backend conventions
└── frontend/       React 19 + Vite + MUI dashboard (consumes the backend)
    AGENTS.md       — frontend conventions
```

## How the pieces fit

```
Postgres (BATCH_* + app tables) <──reads── backend ──serves──> frontend (browser)
```

- **`backend/`** is the dashboard's read-only API. It connects to one or more PostgreSQL / MySQL / Oracle / SQL Server databases (multi-engine, multi-environment) and serves the BATCH\_\* data as JSON. Per-request datasource routing is driven by an `X-Environment` header; per-engine SQL by a routing `SqlDialect`.
- **`frontend/`** is a Vite SPA that authenticates via OAuth2 (one or more configured providers), persists the selected environment in localStorage, and forwards it to the backend on every request.

The dashboard reads from whatever databases are listed under `app.datasources` — point those at any combination of PostgreSQL, MySQL, Oracle, or SQL Server instances that hold Spring Batch metadata you want to inspect.

## Local stack runbook

Order matters the first time (so the schema initdb scripts run before anything connects):

```bash
# 1. Backend — brings up its Postgres via spring-boot-docker-compose
cd backend
cp .env.example .env                      # fill in GITHUB_CLIENT_ID/SECRET
./mvnw spring-boot:run                    # serves on :8080

# 2. Frontend
cd frontend
yarn install
yarn dev                                  # serves on :5173
```

Visit `http://localhost:5173`. Login uses GitHub OAuth — the callback comes back to the **backend**, which sets `JSESSIONID` and redirects to the configured `app.oauth2.success-url` (`/overview` on the frontend).

To work without the backend at all: set `VITE_USE_MOCK_DATA=true` in `frontend/.env` and `yarn dev`. Every API endpoint returns canned data, no requests are made.

## Agent ground rules

These apply to any agent working in this repo. Read first, then dive into the per-component conventions below.

### Response length

- Keep responses concise; prefer code edits with brief summaries over verbose explanation.
- Break long refactors or multi-file changes into smaller focused steps and confirm before proceeding.

### Scope discipline

- Only modify files explicitly mentioned in the request; ask before touching root-level config files (`AGENTS.md`, `CLAUDE.md`, `package.json`, `pom.xml`, etc.).
- Don't scaffold example/template directories or files unless explicitly asked.
- When asked to mirror or sync content, copy only — don't edit the source.

### Build & test verification

- After any multi-file change, run the build (`yarn build` / `./mvnw compile`) and relevant tests before declaring completion.
- For TypeScript changes, verify `tsc --noEmit` passes; for Java changes, verify `./mvnw compile` (or `./mvnw test`) succeeds.
- Fix type errors in test files (missing props/fields after a refactor) before reporting done.

### Database access

- This environment lacks direct PostgreSQL CLI tooling — don't attempt to `psql` / connect directly.
- Either hand back SQL queries for the user to run, or drive the database through Spring Boot's test infrastructure (Testcontainers, `@DataJpaTest`).

## Conventions that span the repo

- **API/UI shapes mirror each other field-for-field.** Backend models in [`backend/.../model/`](backend/src/main/java/com/guavasoft/springbatch/dashboard/model/) and frontend types in [`frontend/src/types/`](frontend/src/types/) (or a page's `types.ts`) are kept in sync; if you rename a field on one side, rename it on the other in the same change.
- **Timestamps are ISO-8601 UTC instants.** Backend converts DB-local timestamps to UTC at the edge using the active datasource's configured `timezone` (defaults to UTC) via [`TimestampFormatter`](backend/src/main/java/com/guavasoft/springbatch/dashboard/config/TimestampFormatter.java); the frontend renders them in the browser's local zone via [`formatTimestamp`](frontend/src/utils/formatTimestamp.ts). Don't emit local strings from new endpoints.
- **`X-Environment` header is sacred.** The frontend sets it from localStorage in an axios interceptor; the backend reads it in `DataSourceContextFilter` to pick the right datasource. Don't add code paths that assume a single datasource or strip the header.
- **Pagination shape is `{ content, page, size, totalElements }`.** Used by `JobRunPage` and `StepDetailPage`. Reuse the shape for any new paginated endpoint.
- **Errors are generic by design.** [`GlobalExceptionHandler`](backend/src/main/java/com/guavasoft/springbatch/dashboard/config/GlobalExceptionHandler.java) never leaks SQL, class names, or stack traces. Don't unwrap exception messages into client responses.
- **Don't write to `BATCH_*` tables from `backend/`.** The dashboard is read-only — those tables are owned by whatever ETL produced the data.
- **80% test coverage is enforced on both sides.** Backend uses JaCoCo (one run boots all three Testcontainers and gates the merged `jacoco.xml` via [`PavanMudigonda/jacoco-reporter`](.github/workflows/pull-request.yml) in CI); frontend uses vitest's `coverage.thresholds` ([`frontend/vite.config.ts`](frontend/vite.config.ts)). Both post sticky PR comments. New code that drops below the threshold fails the workflow — write tests as you go.
- **Pre-commit hook scans for secrets.** [`.pre-commit-config.yaml`](.pre-commit-config.yaml) wires [gitleaks](https://github.com/gitleaks/gitleaks) via the [pre-commit](https://pre-commit.com) framework. One-time per clone: `brew install pre-commit` (or `pipx install pre-commit`) then `pre-commit install`. The `/security-review` skill is the manual deeper-review pass for things regex can't catch (auth flow correctness, debug logs, etc.).

## Where to look for…

- **Adding a new dashboard tile** → start in `frontend/AGENTS.md` (tile components, container/presentation pattern, `useTileQuery`).
- **Adding a new endpoint** → start in `backend/AGENTS.md` (controller/service/repository pattern, JdbcTemplate-vs-JPA decision, error handling).
- **Multi-environment routing** → `backend/AGENTS.md` "Multi-environment / dynamic datasource".
- **Auth flow** → `backend/AGENTS.md` "Security" + `frontend/AGENTS.md` "Authenticated page pattern".

---
> Source: [guavasoftcom/spring-batch-dashboard](https://github.com/guavasoftcom/spring-batch-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
