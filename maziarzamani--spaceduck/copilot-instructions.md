## bun-typescript

> Bun + TypeScript coding conventions for spaceduck


# Bun + TypeScript conventions

## TypeScript

- Strict mode always. No `any`, no `as` casts unless absolutely necessary.
- Prefer `interface` over `type` for object shapes (better error messages, extensibility).
- Use `readonly` on properties that should not change after construction.
- Explicit return types on all exported functions and methods.
- Use barrel exports (`index.ts`) per package — one public API surface.

## Bun specifics

- Use `Bun.env` instead of `process.env` for environment variables.
- Use `Bun.file()` for file reading instead of `fs`.
- Use `Bun.serve()` for HTTP/WebSocket servers.
- Use `Bun.SQL` tagged templates for database queries (not raw `bun:sqlite`).
- Use `bun:test` for all tests (`describe`, `it`, `expect`, `beforeEach`, `afterEach`).
- Use `bun --watch` for dev mode auto-restart.

## Error handling

- Use `Result<T, E>` from `@spaceduck/core` for operations that can fail in expected ways.
- Use `ok(value)` and `err(error)` helper functions to construct results.
- Never throw for expected failures (network errors, validation, missing data).
- Throw only for bugs (programming errors, invariant violations).
- Always include `cause` in error constructors for error chain tracing.

## Imports

- Use workspace imports: `import { X } from "@spaceduck/core"`.
- Relative imports only within the same package.
- No circular imports between packages.

---
> Source: [maziarzamani/spaceduck](https://github.com/maziarzamani/spaceduck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
