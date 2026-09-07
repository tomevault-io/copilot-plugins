## sumamatriculas

> This is an educational math game for children built with Angular 21 (standalone components, SSR) and an Express 5 backend with Passport.js authentication. Local PostgreSQL runs via Docker.

# SumaMatrículas — GitHub Copilot Instructions

## Project Overview

This is an educational math game for children built with Angular 21 (standalone components, SSR) and an Express 5 backend with Passport.js authentication. Local PostgreSQL runs via Docker.

## Tech Stack

- **Frontend:** Angular 21, standalone components, SSR with `@angular/ssr`, signals, no NgModules
- **Backend:** Express 5 (Angular SSR server with custom API routes)
- **Database:** PostgreSQL 17 (Docker, port 5433, user/pass: postgres/postgres)
- **Auth:** Passport.js (Google OAuth + email/password with bcrypt), sessions stored in PostgreSQL
- **Session store:** connect-pg-simple (`session` table)

## Architecture

- The Express server (`src/server.ts`) serves both the Angular SSR app AND a REST API
- API routes are prefixed with `/api/` (auth, profile, operations)
- No Angular Router — view switching is signal-driven conditional rendering in the root template
- No Supabase dependency — all auth and data is local PostgreSQL

## API Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /api/auth/session | No | Check active session |
| POST | /api/auth/login | No | Email/password login |
| POST | /api/auth/register | No | Email/password registration |
| GET | /api/auth/google | No | Google OAuth redirect |
| POST | /api/auth/logout | Yes | Logout |
| GET | /api/profile | Yes | Get user profile |
| POST | /api/profile | Yes | Update profile (age, username) |
| PUT | /api/profile/points | Yes | Update total_points |
| GET | /api/operations | Yes | Get operation history |
| POST | /api/operations | Yes | Log completed operation |

## Code Conventions

- Signals for state, not RxJS subjects
- `@if`/`@for` control flow (no `*ngIf`/`*ngFor`)
- standalone components with inline templates where small
- API calls use plain `fetch()` with `environment.apiUrl` prefix
- CSS: dark glass-morphism theme, Outfit + Inter fonts

## Key Files

- `src/server.ts` — Express server with all API routes + Angular SSR
- `src/app/game.service.ts` — Singleton service with auth, game logic, inactivity monitor
- `src/app/auth.component.ts` — Login/register UI
- `src/app/profile.component.ts` — Stats dashboard with SVG animations

## Running Locally

```bash
docker compose up -d      # Start PostgreSQL
npm start                  # Start app on http://localhost:4000
```

## graphify

For codebase or architecture questions, when `.graphify/graph.json` exists, first run `graphify query "<question>"` (or `graphify path "<A>" "<B>"` / `graphify explain "<concept>"`); these return a scoped subgraph, usually much smaller than `GRAPH_REPORT.md` or raw grep output.
If `.graphify/wiki/index.md` exists, navigate it for deep questions.
If `.graphify/graph.json` is missing but `graphify-out/graph.json` exists, run `graphify migrate-state --dry-run` before relying on legacy state.
Read `.graphify/GRAPH_REPORT.md` only for broad architecture review or when `query` / `path` / `explain` do not surface enough context.
Type `/graphify` in Copilot Chat to build or update the knowledge graph.

---
> Source: [serrig/SumaMatriculas](https://github.com/serrig/SumaMatriculas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
