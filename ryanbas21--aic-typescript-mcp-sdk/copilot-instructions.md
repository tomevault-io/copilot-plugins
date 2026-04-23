## aic-typescript-mcp-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TypeScript SDK for AIC (Authorization Identity Cloud) authentication in AI agents and MCP servers. The SDK enables:
- Token validation against AIC
- 401 responses with IDP authentication instructions
- Token exchange (identity context preservation)

## Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm build

# Lint (ESLint with strict TypeScript rules)
pnpm lint
pnpm lint:fix

# Format (Prettier)
pnpm format
pnpm format:check

# Type check
pnpm typecheck

# Test (Vitest)
pnpm test

# Run single package script
pnpm --filter @anthropic/aic-mcp-sdk build
pnpm --filter @anthropic/mcp-server-example dev

# Run MCP server example (for testing with MCP Inspector)
node apps/mcp-server-example/dist/index.js
```

## Architecture

**Monorepo Structure (pnpm workspaces):**
- `packages/` - Publishable libraries
- `apps/` - Example applications and test servers

**Current Packages:**
- `@anthropic/aic-mcp-sdk` - Core SDK with authentication types and utilities
- `@anthropic/mcp-server-example` - Test MCP server using JSONPlaceholder todos API

**Key Types (packages/aic-mcp-sdk/src/types.ts):**
- `AicConfig` - AIC connection configuration (aicUrl, clientId, clientSecret)
- `TokenValidationResult` - Discriminated union for validation success/failure
- `TokenExchangeResult` - Discriminated union for exchange success/failure
- `AicMcpAuthOptions` - Middleware configuration

## Code Style

- ESM modules (`"type": "module"`)
- Strict TypeScript with `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- Use `readonly` on all interface properties
- Use `type` imports: `import { type Foo } from './foo.js'`
- Use `export type *` for type-only re-exports
- Explicit return types on all functions (`@typescript-eslint/explicit-function-return-type`)
- Prefix unused parameters with `_`
- Numbers in template literals must use `String()`: `` `id: ${String(id)}` ``

## Git Hooks (Lefthook)

Pre-commit hooks auto-fix and stage:
- ESLint fixes on `.ts/.tsx` files
- Prettier formatting on `.ts/.tsx/.json/.yml/.yaml` files

Hooks install automatically via `pnpm install` (prepare script).

## CI/CD

- PR preview packages via pkg-pr-new (`./packages/*`)
- CI runs lint, format check, typecheck, build, and test

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryanbas21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
