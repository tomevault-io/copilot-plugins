## dndmanager

> Hybrid tabletop/computer game platform for Pathfinder 2e. Monorepo with pnpm + Turborepo.

# DnD Manager — Claude Code Conventions

## Project Overview
Hybrid tabletop/computer game platform for Pathfinder 2e. Monorepo with pnpm + Turborepo.

## Tech Stack
- **Frontend:** Next.js 15 (App Router), React Three Fiber, Tailwind CSS, shadcn/ui
- **Backend:** Supabase (PostgreSQL, Realtime, Auth, Storage, Edge Functions)
- **State:** Zustand
- **Testing:** Vitest, Playwright
- **Language:** TypeScript strict throughout

## Monorepo Structure
- `apps/web/` — Next.js frontend
- `packages/pf2e-engine/` — Pathfinder 2e rules engine (pure TS, no UI)
- `packages/game-runtime/` — Game state, turns, fog of war, sync
- `packages/scene-framework/` — Scenario DSL for GM content creation
- `packages/ai-services/` — AI orchestration (Claude, 3D gen, audio)
- `packages/shared/` — Shared types, constants, utilities
- `scenarios/` — GM-created scenarios (Claude Code workspace)
- `supabase/` — Database migrations and config

## Commands
- `pnpm dev` — Start all dev servers
- `pnpm build` — Build all packages
- `pnpm test` — Run all tests
- `pnpm typecheck` — Type check all packages
- `pnpm format` — Format all files

## Rules
- All code must be TypeScript strict
- No `any` types — use `unknown` + type guards when needed
- Prefer named exports over default exports (except Next.js pages)
- Test files go next to source files as `__tests__/` directories
- Database changes require a new migration in `supabase/migrations/`
- PF2e rules must be validated through `@dndmanager/pf2e-engine`
- All game state changes must go through Supabase Realtime
- Server-side dice rolls only (never trust the client)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stefanskiasan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
