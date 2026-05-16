## tony-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack monorepo with a **NestJS** backend (`/api`) and **Angular** frontend (`/ui`). The app manages customers, brokers, files/contracts, and messaging with role-based access control.

## Development Setup

Start the full stack with Docker Compose:

```bash
docker-compose --profile dev up --build   # Development (hot reload)
docker-compose --profile prod up --build  # Production
```

Services: API on port 3000, UI on port 4200, PostgreSQL on 5432, PGAdmin on 5050.

## Common Commands

### Backend (`/api`)

```bash
npm run start:dev     # Start with file watching
npm run build         # Compile TypeScript
npm run lint          # ESLint with autofix
npm run test          # Run unit tests (Jest)
npm run test:watch    # Watch mode
npm run test:cov      # Coverage report
npm run test:e2e      # End-to-end tests
```

### Frontend (`/ui`)

```bash
npm start             # ng serve (dev server on port 4200)
npm run build         # Production build
npm test              # Karma/Jasmine tests
```

To run a single test file in the frontend, use Karma's `--include` flag or focus with `fdescribe`/`fit`.

## Architecture

### Backend (`/api/src`)

Feature-based NestJS modules under `features/`. Shared concerns live in `common/` (guards, decorators, strategies, contracts).

- **Auth**: Passport.js with `LocalStrategy` (login) and `JwtStrategy` (Bearer token). Guards in `common/guards/`.
- **Entities**: TypeORM entities in `entities/`. DB is PostgreSQL 15, `synchronize: true` in dev.
- **Repository pattern**: Feature modules implement interfaces from `common/contracts/`.
- **External integrations**: Firebase Admin (FCM), Puppeteer (PDF generation), Blaudirekt API.

### Frontend (`/ui/projects/shell/src/app`)

Modern Angular 21 with standalone components, no NgModules.

- **State management**: NgRx Signals stores in `data-access/store/`. Each store uses `signalStore()` with `withState`, `withComputed`, `withMethods`, `withHooks`.
- **API services**: In `data-access/provider/` — thin HttpClient wrappers that map to backend endpoints.
- **Auth flow**: `auth.interceptor.ts` injects JWT tokens and handles 401s by refreshing via `AccountStore`. Uses a token refresh queue to avoid concurrent refresh calls. Use `SKIP_AUTH_INTERCEPTION` context token for public endpoints.
- **Routing**: Lazy-loaded standalone components via `loadComponent`. All authenticated routes are nested under `authorized/`.
- **UI**: PrimeNG components + Tailwind CSS (`tailwindcss-primeui`). Rich text via Quill.
- **i18n**: `@ngx-translate` with JSON translation files.

### Dev Proxy

In development, the Angular dev server proxies `/api/*` to `http://tonym_api_dev:3000/` via `ui/proxy.conf.json`. In production, Nginx handles this routing.

## Key Patterns

- Backend modules follow: `module.ts` → `controller.ts` → `service.ts` → repository (implements `common/contracts/` interface).
- Frontend features follow: route → component → store (NgRx Signals) → provider service → API.
- Role-based access uses `@Roles()` decorator on backend controllers and route guards on the frontend.
- TypeScript strict mode is enabled on both sides.

---
> Source: [Draftscode/tony-web](https://github.com/Draftscode/tony-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
