## core-bjesuiter-de

> - **Dev**: `deno task dev` (runs with --watch, auto-reloads on changes to

# AGENTS.md - Codebase Guide for AI Coding Agents

## Commands

- **Dev**: `deno task dev` (runs with --watch, auto-reloads on changes to
  static/ and routes/)
- **Type check**: `deno task type-check` (checks main.ts)
- **Format**: `deno task fmt`
- **Lint**: `deno task lint`
- **Full check**: `deno task check-all` (fmt check + lint + type-check)
- **Build**: `deno task build`
- **DB push (dev)**: `deno task db-push-dev` (Drizzle schema push to Turso dev
  DB)
- **DB push (prod)**: `deno task db-push-prod` (Drizzle schema push to Turso
  prod DB)
- **DB studio (dev)**: `deno task db-studio-dev` (Drizzle Studio for dev DB)
- **No tests configured yet** - check individual files for inline tests or add
  test infrastructure as needed

## Architecture

- **Framework**: Fresh 2.1.2 (Deno web framework with file-based routing)
- **Database**: Turso DB (libSQL) with Drizzle ORM, snake_case columns
  (`@/lib/db/`)
- **UI**: Preact with TailwindCSS 4, WebAwesome components
- **Routes**: File-based in `routes/` (Fresh convention), includes authenticated
  routes in `(authenticated)/` group
- **API**: ts-rest contracts in `contracts/`, Valibot for validation
- **Key libraries**: neverthrow (Result types), date-fns, custom cloudflare api
  client based on ts-rest

## Code Style & Conventions

- **Imports**: Use `@/` alias for absolute imports from workspace root (enforced
  by Cursor rule: avoid `../../` patterns)
- **TypeScript**: Strict types, use Valibot schemas for validation, Drizzle
  types for DB
- **Fresh patterns**: Use `define.page()`, `define.handler()` from
  `@/lib/fresh/defineHelpers.ts`
- **Error handling**: Prefer neverthrow Result types (ok/err) for explicit error
  handling
- **Formatting**: Deno's built-in formatter (run `deno task fmt` before
  committing)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bjesuiter)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bjesuiter)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
