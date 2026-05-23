## village-vandals

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Village Vandals is a full-stack browser strategy game (village builder). Spring Boot 3.5 backend, Vue 3 frontend with PixiJS tile rendering, PostgreSQL database, and optional Keycloak OAuth integration. All services run via Docker Compose.

## Commands

### Backend
```bash
mvn clean package          # Build jar
mvn test                   # Run all tests
mvn test -Dtest=ClassName  # Run a single test class
```
Backend runs on port **8081**. Java 21 with `--enable-preview` is required.

### Frontend
```bash
cd frontend
npm install
npm run dev    # Vite dev server → http://localhost:5173
npm run build  # Production build
npm run format # Prettier formatting
```

### Full Stack (Docker)
```bash
docker compose up   # Starts PostgreSQL, Spring Boot, Vue/Nginx, Keycloak
```

### Environment Variables
Backend reads from env: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `SECRET` (JWT secret).  
Frontend reads `VITE_API_BASE_URL` from `frontend/.env` (default: `http://localhost:8081`).

## Development Process

## Required workflow

For EVERY code change:

1. Add or update automated tests that fail before the change.
2. Implement the fix or feature.
3. Run the relevant test suite.
4. Ensure tests pass before completing.

A task is NOT complete unless:
- behavior is covered by tests
- failing reproduction exists for bug fixes
- new functionality has automated test coverage

Never skip tests because a change seems "small".

### Test-Driven Development
Always write tests when implementing or changing functionality. Follow this order:
1. Write a failing test that captures the requirement
2. Implement the minimum code to make it pass
3. Refactor while keeping tests green

Never deliver a feature or bug fix without accompanying tests.

### Mandatory Spec-Driven Development Flow

**EVERY feature request and bug fix MUST follow this exact flow. No exceptions.**

#### Step 1 — Write a Spec (STOP and wait for approval)
Before any code, produce a spec document covering:
- **Requirements** — what the feature/fix must do, including edge cases and constraints
- **Acceptance criteria** — concrete, testable conditions that define "done"
- **Architecture impact** — packages, schema changes, new endpoints, state management
- **Out of scope** — explicitly state what this change does NOT cover

Present the spec to the user. Do NOT proceed until the user explicitly approves it.
When the spec is approved, save it as `plan/<spec-name>.md` (a single file directly inside the `plan/` root — never a subfolder).

#### Step 2 — Break the Spec into Tasks
Decompose the approved spec into fine-grained tasks using `TaskCreate`. Each task must map to a single verifiable unit of work (one class, method, migration, or UI component). Present the task list for review before starting.
Save each task as its own MD file inside `tasks/<spec-name>/task-N-<short-name>.md`.
Example structure:
```
plan/
  fix-starter-village-resource-production.md   ← spec file
tasks/
  fix-starter-village-resource-production/
    task-1-add-default-rate-constructor.md
    task-2-use-default-constructor-in-village.md
    task-3-write-tests.md
```

#### Step 3 — Implement with TDD
Execute tasks one at a time:
1. Write a failing test that matches an acceptance criterion.
2. Implement the minimum code to pass it.
3. Mark the task complete, then move to the next.

Never begin Step 2 without spec approval. Never begin Step 3 without the task list being complete.

## Architecture

### Auth Flow
1. `POST /auth/login` validates credentials, returns a short-lived JWT + sets an HTTP-only refresh token cookie.
2. Frontend stores JWT in `localStorage` (`jwt_token`). `useSessionStore` (Pinia) manages the `token` and `user` state.
3. Every API request goes through `apiRequest()` in `frontend/src/util/api/api.js`, which injects the JWT as a Bearer token and redirects to `/login` on 401.
4. `JwtAuthFilter` validates tokens on the backend before reaching controllers.
5. `POST /auth/refresh` issues a new JWT using the HTTP-only cookie.
6. Optional Keycloak OAuth2 path: `/auth/callback` exchanges an authorization code for a Keycloak token, then issues an internal JWT.

### Backend Package Structure
Package root: `com.villagevandals.vandals`

| Package | Responsibility |
|---|---|
| `web` | Security config, JWT filter/service, auth endpoints, refresh token |
| `user` | User entity (implements `UserDetails`), registration, `/user` endpoints |
| `village` | Village entity, resource production/storage (embedded value objects), village service |
| `building` | Abstract `Building` base + single-table-inheritance subtypes: `Farm`, `LumberMill`, `Forge`, `Brickyard`, `Barrack` |
| `constructionsite` | Tracks buildings currently under construction per village |
| `resource` | Resource types (food/wood/bricks/iron) and production calculations |
| `app` | World map tile entity and coordinate system |
| `unit` | Military/troop system |
| `gameconfig` | Balance constants (costs, production rates, upgrade formulas) |

**Database schema** is managed by Liquibase (YAML changelogs in `src/main/resources/db/changelog/`). Schema evolves via versioned changesets — never modify the database directly.

Buildings use **Hibernate single-table inheritance** (discriminator column `building_type`). `ResourceStorage` and `ResourceProduction` are **embedded** in `Village` (no separate tables).

Public endpoints (no JWT required): `POST /user/register`, `POST /auth/login`, `POST /auth/callback`.

### Frontend Structure
- **Router** (`frontend/src/router/index.js`): Route guard checks `useSessionStore().isAuthenticated`; routes with `requiresAuth: true` redirect to `/login` if unauthenticated. Auth callback lands at `/auth`.
- **State** (`frontend/src/stores/pinia.js`): `useSessionStore` — persists token to localStorage, syncs across tabs.
- **API layer** (`frontend/src/util/api/`): All backend calls go through `apiRequest()`. Keycloak-specific calls use `keycloak-api.js`.
- **Game rendering**: PixiJS 8 renders the village tile map and building sprites inside Vue components. The world map uses tile coordinates stored in `map_tiles`.

### Key Design Decisions
- **Stateless JWT + HTTP-only refresh cookie**: JWT for stateless REST; refresh cookie for silent renewal without exposing the long-lived token to JS.
- **Single-table inheritance for buildings**: Simplifies queries but means all building columns live in one table.
- **Async resource production**: Villages track production rates in the DB; resource amounts are calculated from elapsed time (checked on fetch, not via a background job per village).
- **CORS**: Allowed origins are `localhost:80` and `localhost:5173` — will need updating for production.

---
> Source: [Frlundsten/village-vandals](https://github.com/Frlundsten/village-vandals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
