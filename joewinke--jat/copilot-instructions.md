## jat

> This is a JST-cloned tenant app. Honor the patterns below.

# Tenant App Conventions

This is a JST-cloned tenant app. Honor the patterns below.

## Stack
- SvelteKit 5 (runes mode — `$state`, `$derived`, `$effect`)
- Tailwind v4 (`@import "tailwindcss"` syntax, NOT v3 `@tailwind base`)
- DaisyUI for components
- Postgres via Drizzle ORM, withRLS wrapper for tenant scoping
- Better Auth for session management

## File layout
- `src/lib/server/` — server-only modules (db, auth, integrations)
- `src/lib/` — shared types, client helpers, components
- `src/routes/` — SvelteKit routes (one folder per route)
- `src/components/` — reusable Svelte components
- `migrations/` — Drizzle SQL migrations, numbered timestamps

## Patterns
- Always wrap DB calls in `withRLS(claims, sql => ...)` for tenant scoping
- Server load functions return typed data, no `any`
- Forms use the JST `<Form>` block, not raw `<form>` HTML
- Auth gates live in `+layout.server.ts`, not inside individual pages

## Forbidden
- Never `console.log` in committed code; use the structured logger
- Never bypass RLS with raw `sql.unsafe`; ask the user first
- Never import from `$lib/server/` inside client-side code

---
> Source: [joewinke/jat](https://github.com/joewinke/jat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
