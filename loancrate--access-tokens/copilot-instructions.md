## access-tokens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A TypeScript monorepo for managing Personal Access Tokens (PATs) backed by DynamoDB. Tokens use scrypt hashing in PHC format with timing-safe comparison. The OAuth 2.0 flow exchanges PATs for short-lived JWTs at `/auth/token`.

## Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build all packages (via Turbo)
pnpm test             # Run unit tests across all packages
pnpm test-int         # Integration tests (requires LocalStack/Docker)
pnpm test:coverage    # Unit + integration tests with coverage merge
pnpm lint             # ESLint across all packages
pnpm typecheck        # TypeScript type checking
pnpm format           # Prettier auto-format
pnpm format:check     # Prettier check (CI enforced)
pnpm verify           # Full CI check: build, lint, typecheck, test, format
pnpm dev              # Run example app (LocalStack required)
```

Single package commands (run from package directory):

```bash
pnpm test                    # Unit tests (Vitest)
pnpm test -- --testNamePattern="MyTest"  # Run specific test
pnpm test-int                # Integration tests (core package only)
pnpm build                   # Build single package
```

Releases use Changesets: `pnpm changeset`, `pnpm version-packages`, `pnpm release`.

## Architecture

**Monorepo structure** (pnpm workspaces + Turborepo):

- **`@access-tokens/core`** — `DynamoDBPat` class: token generation (id62 IDs + random secrets), scrypt hashing, DynamoDB CRUD (issue, verify, revoke, restore, update, list, batchLoad, bootstrap). Schema validation via Zod. Token format: `{prefix}{tokenId}.{base64Secret}`.
- **`@access-tokens/express`** — Express routers and middleware. `createAuthRouter` handles PAT→JWT exchange. `createAdminTokensRouter` provides CRUD endpoints. `createRequireJwt`/`createRequireAdmin`/`createRequireRole` are auth middleware. JWT signing via `jose`. Uses esbuild for bundling + tsc for declarations.
- **`@access-tokens/client`** — `AccessTokensClient` class: type-safe HTTP client for the express API. Uses fetch-retry. Zod schemas for response validation.
- **`@access-tokens/cli`** — Commander-based CLI. Commands: issue, generate, register, list, revoke, restore, update, sync. Supports YAML config files for sync operations.
- **`@access-tokens/example`** — Demo Express app with LocalStack DynamoDB. ESM (`"type": "module"`).
  **Key dependency chain**: cli → client → express → core

## Git Rules

- Never amend commits or force push

## Conventions

- Node.js 20+ required (24.x recommended, see `.nvmrc`); pnpm 10+
- TypeScript strict mode; no `any` types
- Tests co-located in `__tests__/` directories; integration tests use `.int.test.ts` suffix; smoke tests use `.smoke.test.ts`
- Testing with Vitest; globals mode enabled (no imports needed for describe/it/expect); use `vi` from `vitest` for mocking
- DynamoDB roles stored as Sets (enables atomic ADD/DELETE operations); Zod schema transforms Sets to arrays
- Token records validated with Zod on both read and write paths

---
> Source: [loancrate/access-tokens](https://github.com/loancrate/access-tokens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
