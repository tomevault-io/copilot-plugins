## aalma

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AALMA (pilotez la sante mentale au travail) is a SaaS platform for managing workplace mental health through organizational questionnaires, analysis, and action plans. The monorepo contains three projects:

- **`back/`** — NestJS REST API (port 3000)
- **`front/`** — Next.js 16 application dashboard (port 3001)
- **`landing-page/`** — Next.js 16 marketing site

## Development Commands

### Backend (`back/`)
```bash
npm run start:dev          # Dev server with watch (port 3000)
npm run build              # Compile TypeScript
npm run test               # Run Jest unit tests
npm run test:watch         # Tests in watch mode
npm run test:e2e           # End-to-end tests
npm run lint               # ESLint fix
npm run format             # Prettier format
```

### Frontend (`front/`)
```bash
npm run dev                # Dev server (port 3001)
npm run build              # Next.js production build
npm run lint               # Biome linter
npm run format             # Biome formatter
```

### Landing Page (`landing-page/`)
```bash
npm run dev                # Dev server
npm run build              # Next.js production build
npm run lint               # Biome linter
npm run format             # Biome formatter
```

### Database
```bash
docker compose up -d       # Start PostgreSQL 16 container
```
TypeORM migrations are in `back/typeorm/migrations/`. DataSource config is in `back/DataSource.ts`.

## Architecture

### Backend (NestJS)
- **Module-per-feature**: each domain (user, role, company, team, form, question, answer, etc.) is a NestJS module with its own service, controller, and DTOs
- **Auth**: Better Auth library with TypeORM adapter. Cookie-based sessions (`better-auth.session_token`). OAuth providers: Google, Microsoft. Config in `back/src/utils/auth.ts`
- **Authorization**: `@Roles()` decorator + `RolesGuard`. Role IDs: 1=SUPER_ADMIN, 2=ADMIN, 3=CEO, 4=HR, 5=MANAGER, 6=EMPLOYEE, 7=HEALTH_REFEREE
- **Database**: PostgreSQL via TypeORM. 16 entities in `back/typeorm/entities/`. Repositories injected via NestJS DI
- **Activity logging**: `ActivityLogService` tracks all mutations with success/fail status
- **Swagger**: API docs at `/api`
- **Scheduled tasks**: `@nestjs/schedule` for daily notifications (8 AM cron)

### Frontend (Next.js App Router)
- **Route groups**: `(authenticated)` for protected routes, `(main)` for app pages with Sidebar, `(settings)` for settings pages with SettingsSidebar
- **UI**: React Aria Components + Tailwind CSS v4 + Tailwind Variants (`tv()`) for component variants
- **Forms**: React Hook Form with Controller pattern
- **Auth client**: `front/lib/auth-api.ts` wraps auth API calls. Server-side fetch via `front/lib/api.ts` with auto 401/403 redirects
- **Component convention**: page-specific components go in `__components__/` directories next to their page
- **Path alias**: `@/*` maps to `front/*`

### Landing Page (Next.js)
- **Static marketing site** in French — no dynamic data, no API calls
- **Same tech stack** as front: React Aria Components, Tailwind CSS v4, Tailwind Variants, Lucide icons
- **Pages**: Home (sections: Hero, Stats, Features, HowItWorks, Impact, Roles, Team, Trust, CTA) and Contact (demo request form)
- **Environment**: `NEXT_PUBLIC_PLATFORM_URL` points to the front app

### Communication
- Frontend calls backend at `NEXT_PUBLIC_API_URL` (default `http://localhost:3000`)
- Backend CORS configured via `CORS_ORIGIN` env var (default `http://localhost:3001`)
- Landing page links to frontend via `NEXT_PUBLIC_PLATFORM_URL` (default `http://localhost:3001`)

## Code Style
- **Biome** for linting and formatting across all projects — 2-space indentation, double quotes
- Backend additionally uses **ESLint + Prettier**
- Import organization enabled via Biome
- `noExplicitAny` disabled across projects

## Shared UI Patterns
All Next.js projects (front + landing-page) share the same component patterns:
- **React Aria Components** for accessible headless UI primitives
- **Tailwind Variants** (`tv()`) for type-safe variant composition
- **tailwind-merge** for class conflict resolution
- **composeTailwindRenderProps** utility for combining Tailwind + React Aria render props
- **focusRing** reusable focus styling
- **Lucide React** for icons
- **Montserrat** font family
- **Primary color**: orange (#F26700) with shades 40-800

---
> Source: [leanecharpentier/aalma](https://github.com/leanecharpentier/aalma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
