## p4client-ts

> This document provides essential information for AI agents working in the p4-ts repository.

# Agent Guidelines for p4-ts

This document provides essential information for AI agents working in the p4-ts repository.

## Project Overview

A TypeScript monorepo providing typed helpers for the Perforce `p4` CLI. Uses Bun as the runtime and package manager.

**Package Manager:** Bun 1.3.10  
**Runtime:** Node.js >= 18  
**Main Package:** `p4client-ts` (in `packages/core`)

## Build/Lint/Test Commands

All commands run from repo root:

```bash
# Install dependencies
bun install

# Type checking (strict TypeScript)
bun run typecheck

# Build the core package
bun run build

# Run all unit tests
bun run test

# Run a single test file
bun test packages/core/tests/helpers.test.ts

# Run e2e tests
bun run test:e2e

# Full documentation check (builds docs + typechecks API)
bun run docs:check

# Clean build artifacts
bun run clean

# Pack check for npm publishing
bun run pack:core
```

## Code Style Guidelines

### Imports
- Always use `.js` extensions for local imports (e.g., `import { foo } from "./bar.js"`)
- Use `node:` prefix for Node.js built-ins (e.g., `import { spawn } from "node:child_process"`)
- Group imports: external deps first, then internal modules, then types
- Use `import type` for type-only imports

### Formatting
- 2-space indentation
- Single quotes for strings
- Semicolons required
- Max line length: follow existing patterns (~100 chars)

### Types & Naming
- Strict TypeScript enabled (`strict: true`, `noUncheckedIndexedAccess: true`)
- Use `PascalCase` for types, interfaces, and classes
- Use `camelCase` for functions, variables, and properties
- Use `SCREAMING_SNAKE_CASE` for constants
- Prefix private class members with underscore (e.g., `_privateMethod`)
- Avoid `any`; use `unknown` when type is uncertain
- Use explicit return types on public API methods
- Document public APIs with JSDoc comments

### Error Handling
- Create custom error classes extending `Error` for domain-specific failures
- Include structured error categories for classification (see `P4ErrorCategory`)
- Preserve original error context in error properties
- Use pattern matching for error classification from CLI output

### Testing
- Uses Bun's built-in test runner (`bun:test`)
- Test files: `*.test.ts` alongside source or in `tests/` directory
- E2E tests in `tests/e2e/` using test fixtures
- Mock external dependencies (CLI calls) in unit tests
- Use descriptive test names in `describe`/`it` blocks

### Project Structure
```
packages/
  core/           # Main library (p4client-ts)
    src/
      public/     # Public API exports
      internal/   # Implementation details
    tests/
      e2e/        # End-to-end tests
  test-stream/    # Test fixtures
  www/            # Astro Starlight documentation
```

### Key Conventions
- Read-only operations only (no submit, edit, or mutation)
- Preview-first approach (`-n` flag for dry runs)
- **MUST use Effect library** for all async operations and functional patterns
- **MUST use Effect Schema** for runtime validation and data transformation
- JSON output parsing from `p4 -Mj -z tag` commands
- Caching in P4Client instance (respect `refresh` option)
- Always export from `src/public/index.ts` for public API

### Effect Library Guidelines
- Wrap all async operations in `Effect.promise()` or use Effect-based APIs
- Use Effect's error channel for explicit error handling
- Prefer Effect's `pipe()` and `flatMap()` for composable workflows
- Use `Effect.runPromise()` at the application boundary to execute effects

<!-- effect-solutions:start -->
## Effect Best Practices

**IMPORTANT:** Always consult effect-solutions before writing Effect code.

1. Run `effect-solutions list` to see available guides
2. Run `effect-solutions show <topic>...` for relevant patterns (supports multiple topics)
3. Search `~/.local/share/effect-solutions/effect` for real implementations

Topics: quick-start, project-setup, tsconfig, basics, services-and-layers, data-modeling, error-handling, config, testing, cli.

Never guess at Effect patterns - check the guide first.
<!-- effect-solutions:end -->

### Documentation
- JSDoc comments required for all public exports
- Include `@param`, `@returns`, and `@throws` where applicable
- Keep examples practical and relevant to Perforce workflows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peculiarnewbie)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peculiarnewbie)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
