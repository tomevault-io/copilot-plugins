## gym-tracker

> A web application for tracking gym workouts, specifically designed to handle equipment variations across different gyms (locations). It allows users to define programs and routines, and track progress with specific machine variations (Exercises) linked to generic Exercise Types.

# Gym Tracker 2.0

## Project Overview

A web application for tracking gym workouts, specifically designed to handle equipment variations across different gyms (locations). It allows users to define programs and routines, and track progress with specific machine variations (Exercises) linked to generic Exercise Types.

## Tech Stack

- **Framework:** SvelteKit ^2.47, Svelte ^5.41 (Runes)
- **Language:** TypeScript ^5.9 (Strict Mode)
- **State Management:** @tanstack/svelte-query ^6.0 (Svelte Query)
- **Styling:** Tailwind CSS ^4.1 + shadcn-svelte
- **Backend:** Supabase (PostgreSQL)
- **Testing:** Vitest ^4.0

## Architecture: CQRS-Lite

The application separates read (view) and write (command) responsibilities.

`Page → Model → query/mutation → Service`

1.  **Services** (`src/lib/services/`):
    - `XViewService`: Enriched read-only queries (e.g., joins, JSON aggregation).
    - `XCommandService`: Mutations (Create, Update, Delete) mapping to Supabase types.

2.  **Models** (`src/lib/models/*.svelte.ts`):
    - Encapsulate `@tanstack/svelte-query` logic.
    - Expose `CreateQueryResult` and `CreateMutationResult`.
    - Aggregate derived state (loading, error, computations) using Svelte 5 runes (`$state`, `$derived`).

3.  **Context**: Services are injected via `src/lib/context.ts` using Svelte's `setContext`/`getContext`.

## Building and Running

- `npm run check`: Run SvelteKit sync and `svelte-check`.
- `npm run test`: Run tests (Vitest).

## Development Conventions

- **Svelte Query**: Always use helper utils (`fetchQuery`, `updateMutation`, `deleteMutation`) from `$lib/utils/query.ts` and keys from `$lib/query-keys.ts`.
- **Runes**: Use `$state`, `$derived`, and `$effect` (with `untrack` if needed).
- **Strict Types**: No `any`. Use namespace imports for View/Command types (`import * as Workout from '$lib/types/views/workout'`).
- **Database**: All entities use `deleted_at` for soft deletes. View services must filter this out.
- **Rules**: See `.agent/rules/` for detailed coding guidelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zamblauskas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zamblauskas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
