## semrush-mcp

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**semrush-mcp** is a unified Semrush MCP server and CLI for keyword research, domain analytics, backlinks, traffic analysis, and competitive intelligence. It provides two entry points that share the same API client:

- `semrush-mcp` — MCP server (stdio transport) for AI clients (Claude, Cursor, etc.)
- `semrush` — CLI for direct terminal usage

## Commands

```bash
# Development
npm install          # Install dependencies
npm run build        # Build TypeScript to dist/
npm run dev          # Start MCP server in dev mode (tsx)
npm run start        # Start built MCP server

# Testing
npm test             # Run all tests
npm run test:unit    # Unit tests only (mocked, no API key needed)
npm run test:integration  # Integration tests (requires SEMRUSH_API_KEY)

# Quality
npm run lint         # ESLint
npm run format       # Prettier format
```

## Git Hooks

Pre-commit and pre-push hooks are installed via Husky.

**Pre-commit runs:**
- lint-staged (prettier + eslint auto-fix on staged .ts files)
- ESLint (full lint pass)
- TypeScript type-check (`tsc --noEmit`)
- Unit tests (excludes integration tests)
- Secret detection (scans for hardcoded API keys/tokens)

**Pre-push runs:**
- Uncommitted changes check
- Full build (`npm run build`)
- Integration tests (requires `SEMRUSH_API_KEY`)

## Architecture

```
src/
├── index.ts          # MCP server entry point (McpServer API)
├── cli.ts            # CLI entry point (Commander.js)
├── semrush-api.ts    # Shared API client (singleton via lazy Proxy)
├── schemas.ts        # Zod validation schemas + inferred TS types
├── config.ts         # Configuration, logger, env loading
└── __tests__/        # Test suite (unit + integration)
```

**Key patterns:**
- `semrushApi` is a lazy Proxy singleton — import it, never construct `SemrushApiClient` directly (except in tests)
- The API client handles retries (429/5xx), caching, and rate limiting internally
- `SemrushApiError` has `.status` and `.response` fields for error handling

## Environment

Requires `SEMRUSH_API_KEY` in environment or `.env` file.

Optional variables:
| Variable | Default | Description |
|----------|---------|-------------|
| `API_CACHE_TTL_SECONDS` | 300 | Cache TTL for API responses |
| `API_RATE_LIMIT_PER_SECOND` | 10 | Max API requests per second |
| `NODE_ENV` | development | Environment mode |
| `LOG_LEVEL` | info | Logging level |

## ESM Rules

This project uses `"type": "module"` with `"moduleResolution": "NodeNext"`. ALL local imports MUST include the `.js` extension:

```typescript
// CORRECT
import { semrushApi } from './semrush-api.js'
import { DomainParamsSchema } from './schemas.js'

// WRONG — will fail at runtime
import { semrushApi } from './semrush-api'
```

## Coding Conventions

- **Logger**: Writes to `process.stderr` — never use `console.log` in MCP server code (stdout is the MCP transport)
- **CLI output**: Uses `console.log` for user-facing output, `console.error` for errors
- **Error handling**: Catch `SemrushApiError` for API-specific errors; the client retries automatically on 429/5xx
- **Schemas**: All validation schemas live in `src/schemas.ts`; import types via `z.infer<typeof Schema>`
- **No `any`**: ESLint enforces `no-explicit-any: error` in production code (relaxed in tests)
- **Formatting**: Single quotes, no semicolons, 2-space indent, trailing commas (es5)

## Dual Binary

After `npm install -g`, two commands are available:
- `semrush-mcp` — starts the MCP server on stdio
- `semrush` — runs the CLI

Both share the same `SemrushApiClient` and validation schemas.

---
> Source: [mrkooblu/semrush-mcp](https://github.com/mrkooblu/semrush-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
