## sub-tracker

> This document provides context for the Gemini Code Assistant to understand and assist with the development of the **Subscription Tracker** project.

# Gemini Code Assistant Context

This document provides context for the Gemini Code Assistant to understand and assist with the development of the **Subscription Tracker** project.

## Project Overview

The project is a web application designed to help users track their subscriptions. It consists of a **Vue 3** (TypeScript, Vite) frontend and a **Node.js** (Express 5, CommonJS) backend. It supports both **SQLite** and **PostgreSQL** databases, switchable via the `DB_TYPE` environment variable.

Current features:
- Session-based user authentication (single seeded user — no registration)
- User profile management (view, edit, change password)
- Subscription management (CRUD — create, read, update, cancel, delete)
- Dashboard with analytics (monthly/yearly cost totals)
- Separation of active and historical (cancelled) subscriptions with **pagination support**
- Accessible UI components (ARIA-compliant modals with focus management)
- Multi-currency support (Defaulted to **Euro (€)**)
- Docker Compose deployment for both database variants
- **Automated Dual-DB Test Suite:** Comprehensive coverage across SQLite and PostgreSQL.
- **Dedicated Test Setups:** Isolated environments using `vitest.sqlite.js` and `vitest.postgres.js`.
- **Architectural Verification:** Specific tests for DB Factory logic and CSRF token regeneration.
- **Full E2E Coverage:** Cypress tests cover Auth, Dashboard, Profile, and Subscriptions.
- **Standardized Orchestration:** Cross-platform NPM scripts and workspaces replace brittle shell scripts.

## Project Structure

The project is structured as a monorepo using **NPM Workspaces**:

```
/
├── package.json                 # Root manifest with orchestration scripts
├── scripts/                     # Cross-platform Node.js orchestration scripts
├── frontend/                    # Vue 3 + Vite + TypeScript application
│   ├── .dockerignore            # Optimized build context
│   ├── cypress/
│   │   └── e2e/                 # E2E test files
│   └── src/
│       ├── __tests__/           # Vitest component tests
│       └── ...
├── backend/                     # Node.js + Express 5 API
│   ├── .dockerignore            # Optimized build context
│   ├── vitest.sqlite.js         # Dedicated setup for SQLite integration tests
│   ├── vitest.postgres.js       # Dedicated setup for PostgreSQL integration tests
│   ├── eslint.config.mjs        # ESLint 9+ Flat Config (Vitest globals)
│   ├── .prettierrc              # Prettier config
│   └── src/
│       ├── __tests__/           # Backend integration & architectural tests
│       ├── config.js            # Centralized security configuration
│       ├── db/
│       │   ├── sqlite.js        # SQLite adapter (Normalized return shapes)
│       │   ├── postgres.js      # PostgreSQL adapter
│       │   └── ...
├── compose.yaml                 # Docker Compose — SQLite variant
├── compose.postgres.yaml        # Docker Compose — PostgreSQL variant
└── ...
```

## Building and Running

### Docker Compose (recommended)

1.  **Preparation:** `cp .env.example .env` and set `SESSION_SECRET` / `POSTGRES_PASSWORD`.
2.  **Run (SQLite):** `docker compose up --build`
3.  **Run (Postgres):** `docker compose -f compose.postgres.yaml up --build`

## Testing & Quality Assurance

The project enforces high code quality through isolated, environment-aware testing using `npm test` from the root.

### Backend Tests
- **Architectural Isolation:** `src/__tests__/db/factory.test.js` verifies DB switching logic.
- **SQLite:** `npm run test:backend:sqlite` (Uses `vitest.sqlite.js` with per-test file DB isolation and automatic cleanup).
- **PostgreSQL:** `npm run test:backend:postgres` (Handles Docker lifecycle automatically via Node.js orchestration).

### Frontend Tests
- **Linting:** `npm run lint` (ESLint + oxlint + prettier).
- **Unit/Component:** `npm run test:frontend` (Uses non-interactive CI mode `vitest run`).
- **E2E (Cypress):** `npm run test:e2e` (Manual run; excluded from CI and root `npm test` gate).

## Deployment & CI Packages

The CI pipeline is configured to build and push Docker images to **GHCR** on every run.
- **Workflow:** `quality-checks` -> `build-docker-images` -> `backend-test-postgres`.
- **Packages:** Images are available at `ghcr.io/pxl-digital-application-samples/sub-tracker/backend` and `ghcr.io/pxl-digital-application-samples/sub-tracker/frontend`.
- **Caching:** Uses `type=gha` cache backend for GitHub Actions.
- **Versioning:** Images are dual-tagged with `:latest` and the unique `GITHUB_SHA`.
- **Optimization:** Production images use `npm ci --omit=dev` and strict `.dockerignore` for minimal size.

## Development Conventions

-   **Orchestration:** Avoid bash scripts; use Node.js scripts in `scripts/` for complex tasks.
-   **Database abstraction:** All SQL is encapsulated within `backend/src/db/sqlite.js` and `backend/src/db/postgres.js`.
-   **Normalized DB Returns:** All `run()` calls return a consistent `{ changes, rows }` object for adapter interoperability.
-   **Hardened Security:** 
    - Session ID regeneration on login (fixation protection).
    - Centralized `SESSION_SECRET` with production-only mandatory checks.
    - Input length limits on all backend validation rules.
-   **Hardened Lazy Loading:** `sqlite.js` throws an error if accessed while `DB_TYPE` is not `sqlite`.
-   **Health Checks:** The `/api/health` endpoint verifies both API availability and database connectivity.
-   **Test Isolation:** SQLite adapter tests use unique temporary database files (`data/test_adapter_*.db`) to prevent state leakage.
-   **CSRF Security:** Fresh tokens are generated and returned upon successful login to bind to the new session.
-   **Stable Results:** All database queries in tests use `ORDER BY id ASC`.

## API Endpoints
(See README.md for full list)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PXL-Digital-Application-Samples) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
