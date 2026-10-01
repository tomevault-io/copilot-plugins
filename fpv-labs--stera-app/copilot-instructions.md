## stera-app

> This project uses **Ultracite**, a zero-config preset that enforces strict code quality standards through automated formatting and linting.

# Ultracite Code Standards

This project uses **Ultracite**, a zero-config preset that enforces strict code quality standards through automated formatting and linting.

**stera-open** is a Bun monorepo: **Hono** API server, **Drizzle ORM** + **PostgreSQL**, **Better Auth**, **Cloudflare R2** uploads, and a **Flutter** mobile app at `apps/mobile`.

## Quick Reference

- **Format code**: `bun run fix`
- **Check for issues**: `bun run check`
- **Diagnose setup**: `bun x ultracite doctor`

Oxlint + Oxfmt (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use `type` not `interface` for object shapes
- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`
- Named exports; arrow consts for helpers/handlers; `export function createX()` for factories

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security

- Validate and sanitize user input
- Never commit secrets; use `apps/server/.env` (gitignored) and `.env.example` placeholders
- Owner-scope R2 keys as `${userId}/…`

### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports

---

## Monorepo Layout

- `apps/server` — Bun + Hono API (`/api/v1/*`, Better Auth at `/api/auth/*`)
- `apps/mobile` — Flutter app (leave `lib/` conventions to mobile docs)
- `packages/config` — shared TypeScript config
- `packages/env` — validated server env (`@stera/env/server`)
- `packages/db` — Drizzle schema + migrations (`@stera/db`)
- `packages/auth` — Better Auth factory (`@stera/auth`)
- `packages/types` — shared Zod request/response schemas
- `packages/stera_recorder` — Flutter plugin: the AR recorder (Dart + Swift + Kotlin)
- `deploy/` — systemd + nginx configs for EC2

## Testing

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

## When Oxlint + Oxfmt Can't Help

Oxlint + Oxfmt's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Oxlint + Oxfmt can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - API design, data flow, and native module contracts
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Performance and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

Most formatting and common issues are automatically fixed by Oxlint + Oxfmt. Run `bun run fix` before committing to ensure compliance.

---
> Source: [fpv-labs/stera-app](https://github.com/fpv-labs/stera-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
