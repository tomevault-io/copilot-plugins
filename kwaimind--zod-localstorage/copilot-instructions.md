## zod-localstorage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Use `bun` for all commands:

- **Install dependencies**: `bun install`
- **Build**: `bun run build` (builds with tsdown to CJS and ESM formats)
- **Development watch mode**: `bun run dev` (builds on file changes)
- **Run all tests**: `bun run test`
- **Run single test file**: `bun run test <path>` (e.g., `bun run test tests/applyWhere.test.ts`)
- **Typecheck**: `bun run typecheck`
- **Lint**: `bun run lint` (uses Biome)
- **Format**: `bun run format` (uses Biome with --unsafe)

## Architecture

### Purpose
SQL-like query interface for localStorage with Zod schema validation. Provides type-safe querying with select/where operations on arrays.

**IMPORTANT**: All data must be stored as arrays. The schema passed to `ZodLocalStorage` must be a `z.ZodArray` type. This is enforced at compile time.

### Core components

**ZodLocalStorage** (src/index.ts): Main query builder class
- Entry point for all queries
- Only accepts `z.ZodArray<T>` schemas (compile-time enforced)
- Chainable builder pattern with 2 builder classes:
  - `ZodLocalStorage`: Base - provides `selectAll()`, `select()`, `where()`, `execute()`
  - `FilterBuilder`: Handles where filtering on array items
  - `FilterSelectBuilder`: Combines selection and filtering
- Each builder has `.execute()` to run the query
- All operations work on arrays of items

**getFromLocalStorage** (src/getFromLocalStorage.ts): Retrieves and validates data
- Fetches from localStorage by key
- Parses JSON and validates with Zod schema
- Returns null on missing data or parse errors

**applyWhere** (src/applyWhere.ts): Filters individual items based on where clauses
- Supports operators: `=`, `>`, `<`, `>=`, `<=`
- Returns boolean indicating if item matches clause

### Types
- `WhereOperator`: Union of supported comparison operators
- `WhereClause<T>`: Typed object with field, operator, and value

### Testing
Uses Vitest with jsdom environment for localStorage simulation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kwaimind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
