## crawl-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Dev server on port 3666
npm run build        # Production build
npm run test         # Jest (runs in band, 20s timeout)
npm test -- <file>   # Single test file

npm run db:up        # Start PostgreSQL via Docker Compose
npm run db:push      # Apply Prisma schema changes
npm run db:seed      # Seed database
npx prisma studio    # GUI for database inspection
```

## Architecture

SEO crawler SaaS. Users create projects, run crawls (detecting 404s, missing metadata, redirects, noindex issues), review history, and download Excel reports. Includes internal roadmap and CMS modules.

Two coexisting routers:

- Pages Router (`pages/`) - legacy: auth, main dashboard, projects, history, settings
- App Router (`app/`) - new standard: roadmap, CMS, and any new modules going forward

### Module Structure (App Router standard)

All new modules follow this layout (see `docs/app-router-architecture.md`):

```text
app/dashboard/<module>/page.tsx       # Page
app/api/<module>/**/route.ts          # API endpoints
lib/<module>-data.ts                  # Business logic / DB queries
lib/<module>-auth.ts                  # Authentication helpers
lib/server/<module>-access.ts         # Route guards (viewer/editor)
types/<module>.ts                     # DTOs and types
components/<module>/                  # UI components
```

### API Route Conventions

Every `app/api/**/route.ts` must:

- Export `runtime = "nodejs"`
- Wrap handlers with `runPrismaRoute(...)` from `lib/server/prisma-route.ts`
- Throw domain errors with `routeError(status, message)`

`runPrismaRoute` handles standard JSON response shape, Prisma error mapping (`P2025` -> 404, `P2002` -> 409), and eliminates per-route try/catch.

### Authentication

JWT stored in `auth_token` cookie. Token validated in each module's `lib/<module>-auth.ts`. Role-based access via `lib/user-roles.js` - roles: `OWNER`, `SUPER_ADMIN`, `ADMIN`, `EDITOR`, `USER`.

### Database

Prisma + PostgreSQL (Neon). Key models: `User`, `Subscription`, `Project`, `CrawlRun`, `CrawlRunPage`, `RoadmapPhase`, `RoadmapTask`.

Subscription plans: `FREE`, `BASIC`, `STARTER`, `PRO`, `AGENCY` - each with crawl and project limits.

## Design System

Dark-first UI. Fonts: Manrope + Syne (established, do not change).

- Green `#00ff88` - brand/success accent
- Blue `#4d8dff` - interactive/primary actions
- Dense information layout - do not over-space
- Hierarchy through typography (size + weight) before color
- Every visual element must help the user find problems faster - remove decoration that does not serve data

## Working Style

When editing this repository:

1. Inspect the smallest relevant set of files first.
2. Prefer a minimal, targeted fix over broad refactors.
3. Keep Pages Router and App Router boundaries intact unless the task explicitly requires migration.
4. Use `apply_patch` for manual edits.
5. Do not revert unrelated user changes.
6. Validate with the narrowest useful test or build command after the change.

## Content Rules

- Never remove emojis or icons from UI text, labels, or buttons — they are intentional design decisions.
- Never rephrase existing UI text or copy unless explicitly asked to do so.
- Minimum font size is 13px. Never use font sizes below 13px in any UI element.

## Frontend Rules

- Preserve the established visual language: dark-first, Manrope + Syne, compact information density.
- Avoid generic boilerplate layouts and empty decorative UI.
- Make responsive behavior work on desktop and mobile.
- For dashboard and history flows, check route params, hydration order, and session state before fetching data.
- If a fetch depends on authentication, make sure the client session is hydrated before firing the request.

## Debugging Rules

- Read the error path end to end before changing code.
- If a request fails in the browser but works in direct HTTP tests, inspect client state, cookies, and hydration timing.
- If a project or run does not load, verify:
  - `projectId` and `runId` from the URL
  - authenticated user state
  - API response shape
  - cached or delayed client initialization
- When making a user-facing fix, confirm the affected page still builds successfully.

## Output Expectations

- State the files changed.
- State the validation performed.
- If something was not verified, say so explicitly.

---
> Source: [FeMizo/Crawl-Site](https://github.com/FeMizo/Crawl-Site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
