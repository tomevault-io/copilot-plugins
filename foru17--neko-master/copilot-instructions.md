## neko-master

> Guidance for coding agents working in this monorepo.

# AGENTS.md

Guidance for coding agents working in this monorepo.

## Scope and Priorities

1. Follow this file for repo-specific workflows and conventions.
2. Follow direct user instructions over this file when they conflict.
3. Keep changes minimal, targeted, and consistent with existing code.

## Repository Overview

- Package manager: `pnpm` (workspace monorepo).
- Task runner: `turbo` at repo root.
- Main apps:
  - `apps/web`: Next.js 16 + React 19 frontend.
  - `apps/collector`: Fastify + WebSocket + SQLite backend service.
- Shared package:
  - `packages/shared`: shared TypeScript types and utility functions.

## Environment and Toolchain

- Node.js: project docs target Node 22.
- TypeScript is strict across packages.
- Module systems:
  - `apps/collector` and `packages/shared`: ESM (`type: module`, `NodeNext`).
  - `apps/web`: Next.js/bundler TS config.

## Install and Bootstrap

- Install dependencies: `pnpm install`
- Start all apps in dev mode: `pnpm dev`
- Optional helper script: `./start.sh`

## Build, Lint, and Test Commands

### Root (all workspaces)

- Build all packages/apps: `pnpm build`
- Run lint across workspaces: `pnpm lint`
- Start all dev tasks: `pnpm dev`

### Filtered workspace commands (recommended during development)

- Build web only: `pnpm --filter @neko-master/web build`
- Build collector only: `pnpm --filter @neko-master/collector build`
- Build shared only: `pnpm --filter @neko-master/shared build`
- Lint web only: `pnpm --filter @neko-master/web lint`
- Lint collector only: `pnpm --filter @neko-master/collector lint`

### Tests

- This repo currently has tests in `apps/collector` (Vitest).
- Run all collector tests: `pnpm --filter @neko-master/collector test`
- Watch mode: `pnpm --filter @neko-master/collector test:watch`

### Run a single test file (important)

- Run one test file:
  - `pnpm --filter @neko-master/collector test -- src/modules/auth/auth.service.test.ts`
- Run one test file by pattern:
  - `pnpm --filter @neko-master/collector test -- src/**/*.geoip-config.test.ts`
- Run a single test case by name:
  - `pnpm --filter @neko-master/collector test -- -t "should validate token format"`
- Run single file + test name:
  - `pnpm --filter @neko-master/collector test -- src/modules/auth/auth.service.test.ts -t "should validate token format"`

### Writing tests (collector)

- Use Vitest (`describe`/`it`/`expect` imported from `'vitest'`).
- Use `createTestDatabase()` and `createTestBackend()` from `src/__tests__/helpers.ts` to set up a temporary SQLite DB in `beforeEach` and call `cleanup()` in `afterEach`.
- Test files live next to the code they test (e.g., `auth.service.test.ts` beside `auth.service.ts`).

### Type checks and push safety

- Collector type check: `pnpm --filter @neko-master/collector exec tsc --noEmit`
- Note: `.husky/pre-push` runs checks when pushing to `main`:
  - collector `tsc --noEmit`
  - web `build`

## Code Style Guidelines

### General TypeScript Rules

- Prefer explicit types on exported APIs and `import type` for type-only imports.
- Keep strict-null and unknown-safe patterns; narrow with guards.
- Avoid `any`; if unavoidable, keep scope tiny and document briefly.
- Prefix intentionally unused variables/parameters with `_` (e.g., `_unused`).
- Prefer small pure functions for transformations and parsing logic.

### ESLint Key Rules (collector)

- `no-console`: **`console.log` is forbidden** — use `console.info`, `console.warn`, or `console.error` instead.
- `@typescript-eslint/no-unused-vars`: warn-level; variables/args prefixed with `_` are exempted.
- `@typescript-eslint/no-explicit-any`: warn-level in source code.
- **Test files** (`*.test.ts`, `__tests__/**`): `no-explicit-any` and `no-console` are turned off.

### Imports and Module Boundaries

- Keep import groups logically ordered:
  1) platform/native modules,
  2) third-party packages,
  3) workspace/internal modules.
- Use relative imports with `.js` extension in ESM NodeNext packages (`collector`, `shared`).
- In web app code, prefer alias imports via `@/*` for internal modules.
- Do not create cross-app deep imports; use `@neko-master/shared` for shared contracts.

### Formatting and Layout

- Match local file style instead of forcing global formatting changes.
- Existing style is **not uniform**, even within a single package:
  - `apps/web`: double quotes and semicolons.
  - `apps/collector`: mixed — some files use single quotes, others use double quotes. Always follow the style of the file you are editing.
  - `packages/shared`: single quotes and semicolons.
- Preserve existing whitespace and line-wrap style in touched files.

### Naming Conventions

- Filenames: kebab-case (e.g., `auth.service.ts`, `use-gateway.ts`).
- React components: PascalCase exports; file names remain kebab-case.
- React hooks: `useXxx` naming.
- Types/interfaces/enums: PascalCase.
- Variables/functions: camelCase.
- Environment variables/constants: UPPER_SNAKE_CASE.
- Database-origin fields may use snake_case (`is_active`, `created_at`); preserve API contracts.

### Error Handling and Logging

- In `catch`, use `error: unknown`, then narrow via `instanceof Error`.
- Return structured API errors for HTTP handlers (e.g., `{ error: string }`) with proper status codes.
- Add contextual logs for operational failures (network, DB, background jobs).
- Use timeouts for external network calls where appropriate (`AbortSignal.timeout(...)`).
- Prefer resilient flows (`Promise.allSettled`) when partial results are acceptable.

### Backend (collector) Patterns

- Keep business logic in services; route/controller handlers should stay thin.
- Reuse repository/base-repository helpers for DB query logic.
- Maintain backend isolation by `backendId` in data paths and caches.
- For auth/security behavior, preserve cookie/token compatibility paths.
- **Fastify plugin pattern**: controllers export a `FastifyPluginAsync` function, registered in `app.ts` via `app.register(controller, { prefix: '/api/xxx' })`. Services are injected onto the Fastify instance through `declare module 'fastify'` augmentation and accessed as `fastify.xxxService`.

### Frontend (web) Patterns

- Respect Next.js App Router conventions (`app/[locale]/...`).
- Add `"use client"` only for client components/hooks that need it.
- Prefer React Query hooks for server state and caching.
- Keep API contract types aligned with `@neko-master/shared` and `apps/web/lib/api.ts`.
- Reuse shared utility helpers (`cn`, formatting utilities) instead of ad-hoc duplicates.
- **i18n**: uses `next-intl` with two locales (`zh` default, `en`).
  - Translation files: `apps/web/messages/{zh,en}.json`.
  - Routing config: `apps/web/i18n/routing.ts`.
  - Use `useTranslations('namespace')` in components; never hardcode user-facing strings.
- **UI components**: shadcn/ui (`new-york` style) + Tailwind CSS v4 + `lucide-react` icons.
  - Base UI primitives live in `apps/web/components/ui/`.
  - Config: `apps/web/components.json`.
  - Always reuse existing UI components before creating new ones.

### Shared Package Patterns

- `packages/shared` should stay framework-agnostic and side-effect free.
- Keep shared types backward compatible when possible.
- Export shared helpers from `packages/shared/src/index.ts` intentionally.

## Testing Expectations for Changes

- Backend-only changes: run collector lint + targeted tests first, then full collector tests if broad.
- Frontend-only changes: run web lint; also run web build for production-impacting changes.
- Cross-package changes: run `pnpm lint` and `pnpm build` before final handoff when feasible.

## Cursor/Copilot Rules Check

- Checked for Cursor rules:
  - `.cursor/rules/` -> not found
  - `.cursorrules` -> not found
- Checked for Copilot instructions:
  - `.github/copilot-instructions.md` -> not found
- If any of these files are added later, treat them as additional mandatory guidance.

## Agent Working Notes

- Prefer minimal, surgical edits over large refactors.
- Avoid renaming/moving files unless required by the task.

---
> Source: [foru17/neko-master](https://github.com/foru17/neko-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
