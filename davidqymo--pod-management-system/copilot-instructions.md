## pod-management-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

### Backend (Maven multi-module — `backend/`)
```bash
# Full build (compile + unit tests + integration tests)
cd backend && mvn clean verify

# Run unit tests only
cd backend && mvn test

# Run integration tests only (Testcontainers — requires Docker)
cd backend && mvn verify -DskipUnitTests

# Run a single test class
cd backend && mvn test -Dtest=ResourceServiceTest

# Run a single test method
cd backend && mvn test -Dtest=ResourceServiceTest#testChangeStatus_activeToInactive

# Run the app (needs PostgreSQL + Redis running)
cd backend && mvn spring-boot:run

# Skip tests during build
cd backend && mvn clean install -DskipTests
```

### Frontend (Vite + npm — `frontend/`)
```bash
cd frontend && npm ci          # install dependencies
cd frontend && npm run dev     # dev server at http://localhost:5173 (proxies /api → localhost:8080)
cd frontend && npm run build   # production build (tsc + vite)
cd frontend && npm run lint    # ESLint
cd frontend && npm test        # Vitest (watch mode)
cd frontend && npm run test:coverage  # Vitest with coverage
```

### E2E Tests (Playwright — `frontend/e2e/`)
```bash
cd frontend/e2e && npm ci
cd frontend/e2e && npx playwright install --with-deps chromium
cd frontend/e2e && npm run test:e2e
```

### Root-level build
```bash
mvn clean verify   # builds both backend and frontend Maven modules
```

## Architecture

**Stack:** Java 17 + Spring Boot 3.2.5 + JPA/Hibernate + PostgreSQL | React 18 + TypeScript + Vite 5 + Tailwind | Redis (caching/distributed locks) | Flyway (migrations)

**Three-tier backend:** Controller → Service → Repository (JPA). Frontend is component-driven with React Query for data fetching.

### Backend package structure (`com.pod.*`)
- `controller/` — REST endpoints (currently `ResourceController`, `RateController`)
- `service/` — Business logic with pessimistic locking (`ResourceService`, `RateService`)
- `repository/` — Spring Data JPA with custom queries and `@Lock(PESSIMISTIC_WRITE)`
- `entity/` — JPA entities + enums (`Resource`, `Rate`, `CostCenter`, `Holiday`, `Skill`, `User`, `ResourceCategory`, `ResourceStatus`)
- `exception/` — Domain exceptions (`InvalidStatusTransitionException`, `RatePeriodGapException`)

### Frontend structure (`frontend/src/`)
- `components/`, `pages/`, `hooks/`, `services/` (API calls), `types/`, `utils/`
- `__tests__/` — Vitest + Testing Library (jsdom)
- Path alias: `@` → `./src`
- Vite dev server proxies `/api` requests to `http://localhost:8080`

### Key libraries
- **Lombok + MapStruct** — entity boilerplate and DTO mapping (annotation processors configured in `maven-compiler-plugin`)
- **ShedLock** — distributed lock for scheduled tasks (Redis-backed)
- **Jackson JSR-310** — Java 8 date/time serialization
- **frappe-gantt** — Gantt chart rendering
- **Recharts** — dashboard charts
- **TanStack React Query** — server state management

### Database
- Flyway migrations in `backend/src/main/resources/db/migration/`
- `V1__initial_schema.sql` → core tables; `V2__add_audit_triggers.sql` → audit logging
- Seed data: `V1.1` (cost centers), `V1.2` (holidays), `V1.3` (test users), `V1.4` (skills)
- **Soft delete everywhere** — `is_active` flag, no hard deletes; reversal pattern for corrections

### Test setup
- **Backend:** JUnit 5 + Mockito + Testcontainers (PostgreSQL 15 + Redis 7). H2 available as fallback for unit tests without Docker. JaCoCo enforces 80% line coverage. Test patterns: `*Test.java` (unit), `*IT.java` (integration).
- **Frontend:** Vitest (jsdom) + Testing Library + Playwright E2E. E2E tests in separate `frontend/e2e/` directory with its own `package.json`.

## Domain Constraints

These are hard business rules enforced in the service layer — violating them causes domain exceptions:

- **Rate periods** must be contiguous (no gaps). `effective_to` is auto-calculated as next rate's `effective_from` minus 1 month. Overlapping periods are rejected.
- **Resource status transitions** follow a state machine (`ResourceStatus` enum). Invalid transitions throw `InvalidStatusTransitionException`.
- **5-project monthly cap** — a resource cannot be allocated to >5 distinct projects in any calendar month.
- **144 hours/month** per HCM; 36h/month OT cap; daily 8+2 regular/OT split.
- **Allocation semantics** — override (replace), not accumulate. New allocation for (resource, project, week) replaces existing.
- **Approval gate** — all allocations require POD Manager approval.
- **Import idempotency** — CSV imports tracked by `import_batch_id`; duplicates silently skipped.
- **Budget unit** — K USD with 2 decimal places (e.g., `4.95` = $4,950).
- **Optimistic locking** — `version` column on allocations; concurrent edits produce conflict errors.

## Coding Standards

- **Naming:** Java camelCase / PascalCase / SCREAMING_SNAKE_CASE. DB tables: `snake_case` plural. Package: `com.pod.*`.
- **Style:** Google Java Style Guide (4-space indent). ESLint + Prettier for frontend.
- **Comments:** JavaDoc on public classes/methods; inline comments only for non-obvious logic.
- **Commit format:** Conventional commits (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`).

## Key Reference Files

- `doc/PRODUCT_REQUIREMENTS.md` — single source of truth for requirements (v1.5)
- `doc/TECHNICAL_DESIGN.md` — technical design document (v2.3)
- `doc/TECHNICAL_IMPLEMENTATION_PLAN.MD` — sprint implementation plan
- `PROGRESS.md` — current project status and completed milestones

## Workflow

- **Superpowers:** Use `superpowers:brainstorming` before design work; `superpowers:writing-plans` before implementation; TDD discipline (test → minimal code → refactor).
- **Branch model:** `main` (production), `develop` (integration), `feature/*`, `hotfix/*`. Merge via PR to `develop`.
- **YAGNI:** Defer Phase 2 features (Actual Consumption UI, What-if Gantt, Skill-matching) until MVP validated.

---
> Source: [davidqymo/POD-Management-System](https://github.com/davidqymo/POD-Management-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
