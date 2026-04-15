## trigger-adapters

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Turborepo monorepo for `trigger-adapters`, a library that provides framework-specific adapters for Trigger.dev SDK integration. The project uses Bun as the package manager and Biome for linting/formatting.

## Repository Structure

- **`packages/trigger-adapters/`** - Main library package with adapter implementations
  - `/src/core.ts` - Core trigger function that wraps `@trigger.dev/sdk`
  - `/src/express.ts` - Express.js adapter
  - `/src/hono.ts` - Hono adapter
  - `/src/nextjs.ts` - Next.js adapter
- **`apps/www/`** - VitePress documentation site
- Root uses Turborepo for monorepo management with workspace configuration in `package.json`

## Development Commands

### Root Commands (Monorepo)
```bash
# Install dependencies
bun install

# Build all packages
bun run build
# or
turbo run build

# Development mode (all packages)
bun run dev
# or
turbo run dev

# Run Biome checks (linting & formatting)
bun run check

# Run Biome checks with auto-fix
bun run check:write

# Type checking across all packages
bun run check-types
# or
turbo run check-types

# Build/dev specific package
turbo run build --filter=trigger-adapters
turbo run dev --filter=www
```

### Package-specific Commands (`packages/trigger-adapters/`)
```bash
# Navigate to package directory
cd packages/trigger-adapters

# Build the library (uses tsdown)
bun run build

# Development mode with watch
bun run dev

# Run tests with Vitest
bun run test

# Type checking
bun run typecheck

# Release new version (uses bumpp)
bun run release
```

### Documentation Site (`apps/www/`)
```bash
cd apps/www

# Start dev server
bun run dev

# Build documentation
bun run build

# Preview built documentation
bun run preview
```

## Code Architecture

### Trigger Adapters Library

The library provides a unified interface for integrating Trigger.dev with different web frameworks. Each adapter:

1. **Core Function** (`core.ts`): Wraps the Trigger.dev SDK's `tasks.trigger()` function
2. **Framework Adapters**: Each adapter exports a `handler()` function that:
   - Extracts the task ID from the request (typically from `params.id`)
   - Extracts the payload from the request body
   - Calls the core `trigger()` function
   - Returns the result in the framework-appropriate format

The package is built with:
- **tsdown** for bundling (outputs both ESM and CJS)
- Exports configured in `package.json` for tree-shaking support
- TypeScript with strict mode and module preservation

### Code Style & Formatting

- **Biome** is used for linting and formatting (configured in `biome.json`)
- Tab indentation
- Double quotes for JavaScript strings
- Auto-organize imports enabled
- All Biome recommended rules are enabled

### Testing

- **Vitest** is used for testing the trigger-adapters package
- Test files are located in `packages/trigger-adapters/tests/`
- Run tests with `bun run test` from the package directory

### Build System

- **Turborepo** manages the monorepo build pipeline
- Build outputs:
  - trigger-adapters: `dist/` directory with `.js`, `.cjs`, and `.d.ts` files
  - www (VitePress): `.vitepress/dist/` directory
- Tasks are configured in `turbo.json` with dependency management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackall3n)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jackall3n)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
