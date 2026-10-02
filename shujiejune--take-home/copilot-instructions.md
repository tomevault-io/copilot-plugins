## take-home

> > Guidance for AI agents working in this repository. Read this first.

# AGENTS.md

> Guidance for AI agents working in this repository. Read this first.

## Project

e-commerce: Take-home full-stack product catalog and ordering service.

## Stack

- Languages: Java 17; TypeScript for the planned frontend
- Runtime / package managers: JVM + Maven Wrapper; Node.js + npm for the planned frontend
- Backend: Spring Boot 4.1.1
- Frontend: React 19.3 + Vite (planned; not yet scaffolded)
- Data: PostgreSQL + Flyway (planned)
- Packaging: Docker and Docker Compose

## Getting Started

The repository currently contains only the initial backend scaffold:

```bash
cd backend
./mvnw test
./mvnw spring-boot:run
```

The checked-in `backend/compose.yaml` currently defines no services. Follow `docs/TDD.md` and `docs/TODO.md` when adding PostgreSQL, the React frontend, and root container packaging.

## Commands

Run backend commands from `backend/`.

- Build: `./mvnw package`
- Test: `./mvnw test` (single class: `./mvnw -Dtest=BackendApplicationTests test`)
- Lint / format: No dedicated command is configured

Run frontend commands from `frontend/` (see `frontend/README.md` for details):

- Install: `npm ci`
- Dev server: `npm run dev` (proxies `/api` to the backend; `VITE_USE_MOCK=true` serves the in-memory demo backend instead)
- Test: `npm test`
- Build: `npm run build`

## Layout

- `backend/src/main/java/`: application source under `com.example.backend`
- `backend/src/main/resources/`: Spring Boot configuration and future Flyway migrations
- `backend/src/test/java/`: backend tests
- `backend/compose.yaml`: generated local service file (currently empty; root Compose packaging is planned)
- `frontend/`: React consumer page (product catalog and idempotent order form; see `frontend/README.md`)
- `docs/Requirements.md`: normalized functional and non-functional requirements
- `docs/TDD.md`: approved architecture and implementation design
- `docs/API.md`: human-readable HTTP behavior contract
- `docs/openapi.yml`: machine-readable OpenAPI 3.1 contract
- `docs/Database.md`: PostgreSQL schema, query, and transaction contract
- `docs/TODO.md`: milestone checklist and implementation progress
- `Take-Home_Assignment.docx`: original assignment specification

## Conventions

- Follow standard Java, TypeScript, Spring Boot, and React naming conventions.
- Keep backend modules organized by feature (`product`, `order`, `shipping`, `security`) with API, application, domain, and persistence responsibilities separated as described in `docs/TDD.md`.
- Use Java records for API DTOs, normal classes for JPA entities, and sealed types only for genuinely closed outcome hierarchies.
- Keep `docs/API.md` and `docs/openapi.yml` synchronized whenever an HTTP contract changes.
- Make schema and seed-data changes through Flyway migrations that follow `docs/Database.md`; do not rely on automatic schema generation.
- Keep PostgreSQL as the source of truth for inventory. Do not introduce Redis locks, caching, a broker, or an outbox without revisiting `docs/TDD.md`.
- Add or update tests for behavior changes, using PostgreSQL-backed integration tests for transaction and concurrency behavior.
- Use the Maven Wrapper rather than assuming a system Maven installation; commit the frontend lockfile and use `npm ci` once the frontend exists.
- Root `AGENTS.md` governs the whole repository. Add module-level files only if a module develops materially different commands or conventions, and do not duplicate root guidance.
- **Commits** follow Conventional Commits: `action(field): content` (e.g. `feat(order): add delivery quote endpoint`, `fix(auth): handle expired JWT`). Imperative mood, lowercase, no trailing period.
  - `field` is the module/area touched.
  - Every implementation task that changes files MUST end with a git commit before the final response. Stage only files/hunks that belong to the current task; never bundle unrelated changes. Read-only tasks do not create empty commits.
- **Before editing**: inspect `git status` and treat pre-existing or concurrent changes as user-owned.
- **After editing**: review the final diff and run proportionate verification before committing.

## Constraints

- Do not edit generated output under `backend/target/`, `frontend/dist/`, or `frontend/node_modules/`.
- Do not commit IDE metadata, build artifacts, credentials, local secrets, or a real API key.
- Treat `Take-Home_Assignment.docx` as source material; do not modify it unless explicitly requested.

---
> Source: [shujiejune/take-home](https://github.com/shujiejune/take-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
