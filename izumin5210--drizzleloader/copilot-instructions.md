## drizzleloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

drizzleloader is a code generation tool that automatically generates DataLoader instances from Drizzle ORM database schemas. It eliminates boilerplate code for implementing data-loading patterns in GraphQL applications.

## Commands

```bash
pnpm build          # Compile TypeScript (tsc)
pnpm test           # Run Vitest test suite
pnpm test <pattern> # Run specific test file
pnpm biome check    # Lint and format check
pnpm biome check --write # Auto-fix lint and format issues
```

## Architecture

The codebase follows a pipeline architecture:

1. **Analyzer** (`src/analyzer/`) - Inspects Drizzle table definitions to extract loader metadata (primary keys, indexes)
2. **Generator** (`src/generator/`) - Generates TypeScript DataLoader code from analyzed tables
3. **CLI** (`src/index.ts`) - Entry point using Commander.js, orchestrates the analysis → generation flow

Key utilities:
- `src/utils/naming.ts` - Case conversion (snake_case → PascalCase/camelCase)
- `src/utils/type-mapping.ts` - Drizzle column types → TypeScript types

## Generated Loader Pattern

The generator produces loaders with two patterns:
- **Unique loaders** (primary key, unique indexes): Returns single value or throws `DrizzleLoaderNotFound`
- **Non-unique loaders** (regular indexes): Returns arrays

Composite indexes and conditional indexes (with WHERE) are intentionally skipped.

## Technical Stack

- ES Module (`"type": "module"`)
- TypeScript strict mode with `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- Biome for linting/formatting (double quotes, 2-space indent)
- Vitest for testing
- Peer dependencies: `dataloader@^2.0.0`, `drizzle-orm@>=0.30.0`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izumin5210) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
