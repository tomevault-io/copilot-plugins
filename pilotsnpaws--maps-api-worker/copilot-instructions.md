## maps-api-worker

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a Cloudflare Worker that provides a REST API for interacting with a MySQL database via Cloudflare Hyperdrive. It supports PNP trip and volunteer mapping pages by serving trip data from a MySQL database.

## Essential Commands

### Development
```bash
npm run dev              # Start local development server on http://localhost:8787/
npm start                # Alias for npm run dev
```

### Testing
```bash
npm test                 # Run tests with Vitest
```

### Deployment
```bash
npm run deploy           # Deploy to Cloudflare Workers
```

### Type Generation
```bash
npm run cf-typegen       # Regenerate worker-configuration.d.ts from wrangler.jsonc bindings
```

Run this after modifying bindings in `wrangler.jsonc` to update TypeScript types.

## Architecture

### Single-File Worker
This is a minimal Cloudflare Worker with all logic in `src/index.ts`. There are no separate route handlers, middleware, or utility modules.

### Database Connection via Hyperdrive
The worker uses Cloudflare Hyperdrive (configured in `wrangler.jsonc`) to connect to MySQL:
- **Binding name**: `HYPERDRIVE` (accessed via `env.HYPERDRIVE` in the worker)
- **Local development**: Set `WRANGLER_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE` environment variable for local MySQL connection
- **Production**: Uses Hyperdrive ID `2d8371c752954d879f96d49819ce014e`

Database connections are created per-request and closed via `ctx.waitUntil(connection.end())`.

### API Structure
Currently implements a single endpoint:
- **GET /trips**: Returns trip data from `prod_forum.vw_lines` view with optional filters:
  - `last_post_before`: Filter trips before date
  - `last_post_after`: Filter trips after date  
  - `updated_last_days`: Filter trips updated within N days (defaults to 3 if no filters provided)

Query parameters are converted to SQL WHERE clauses with parameterized queries.

### Testing Setup
Tests use `@cloudflare/vitest-pool-workers` for Cloudflare Workers environment simulation. The test file imports `cloudflare:test` module which provides:
- `env`: Mock environment variables/bindings
- `createExecutionContext()`: Create execution context for `ctx` parameter
- `waitOnExecutionContext()`: Wait for `ctx.waitUntil()` promises before assertions
- `SELF`: Integration-style testing via fetch

Note: Current tests in `test/index.spec.ts` are boilerplate and don't reflect actual `/trips` endpoint behavior.

## Configuration Files

- **wrangler.jsonc**: Worker configuration including Hyperdrive binding (uses JSONC format with comments)
- **tsconfig.json**: Main TypeScript config (strict mode enabled, targets ES2021, excludes test directory)
- **test/tsconfig.json**: Extends main config with experimental Cloudflare Workers types for testing
- **worker-configuration.d.ts**: Auto-generated from `wrangler.jsonc` bindings via `npm run cf-typegen`

## Key Dependencies

- **mysql2**: MySQL client library (requires v3.13.0+ for Cloudflare Workers compatibility)
- **wrangler**: Cloudflare Workers CLI for dev/deploy
- **vitest**: Test runner with `@cloudflare/vitest-pool-workers` for Workers-specific testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pilotsnpaws)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pilotsnpaws)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
