## octto

> - Never use em dashes. Use colons for definitions, commas or parentheses for asides, and restructure sentences that rely on em dashes.

# Octto: Project Rules

## Writing

- Never use em dashes. Use colons for definitions, commas or parentheses for asides, and restructure sentences that rely on em dashes.

## Code Style

- No classes for business logic. Use factory functions (`createX`) with closed-over state
- No nesting beyond 2 levels inside a function body. Prefer early returns and small helpers
- Max function length: 40 lines (skipBlankLines, skipComments)
- No magic numbers/strings. Use named constants. Place shared tunables in `constants.ts`
- No `any` types, no type assertions (`as Type`). Use Zod/Valibot schemas or type guards to narrow
- Use `unknown` at system boundaries and normalize with schemas before handling
- No comments explaining *what*, only *why* when non-obvious
- Double quotes, semicolons, trailing commas (enforced by Biome)

## Architecture

- Plugin entry point (`src/index.ts`) exports default `Plugin`
- Keep modules focused: `session/` for lifecycle, `tools/` for MCP tools, `state/` for persistence, `hooks/` for prompt injection
- Use named exports only in `src` (except the plugin entry point)
- Re-export public APIs through barrel files (`index.ts`)

## TypeScript

- Names are contracts: domain-meaningful, no `data`/`result`/`temp`
- Prefer single-word names. Drop redundant prefixes. Context (scope, parameter position, containing object) should carry the qualifier
- No type names in identifiers (no Hungarian notation): avoid suffixes like `Map`, `Array`, `List`, `String`, `Object`, `Set`, `Dict`, `Number`, `Boolean`, `Fn`, `Func`, `Callback`
- Prefer `interface` for contracts and `type` for unions/aliases
- Discriminated unions over class hierarchies
- Use `as const` constant maps for statuses/events and derive union types from them
- Use `import type` for type-only imports
- Explicit return types on exported functions
- `readonly` on data structures that shouldn't mutate

## Module Structure

- Order files as: imports -> exported types/constants -> internal constants/schemas -> main factory -> private helpers
- Keep comments sparse and only for non-obvious behavior

## Imports and Paths

- Use `@/*` aliases for cross-folder project imports
- Use `./` relative imports within the same folder
- No parent-relative imports (`../`) where `@/*` is appropriate

## Engineering Principles

- DRY: extract shared patterns, no copy-paste
- YAGNI: no speculative features or unused abstractions
- Fail fast: validate inputs early, return/throw before the happy path
- Dependency injection: pass dependencies in, don't import singletons
- Errors are values: custom error types with context, no bare `catch {}`

## Testing

- Test real behavior, not mocked behavior. If a mock is the only thing being verified, the test is wrong
- Mock data, not behavior. Inject test data, don't spy on implementation details
- All error paths must have tests
- All public exports must have tests
- Test output must be pristine. Capture and validate expected errors
- Place tests in `tests/*.test.ts` with behavior-focused `it(...)` names
- Use `/tmp` unique paths for filesystem tests and always cleanup in `afterEach`
- Prefer condition polling helpers (`waitUntil` style) over fixed sleeps
- Use Bun's built-in test runner (`bun test`)

## Tooling

- `bun run check` runs the full quality gate: `biome check . && eslint . && tsc --noEmit && bun test`
- Pre-commit hook runs Biome check + ESLint fix on staged files via lefthook
- CI runs full `bun run check` on every PR
- Run `bun run check` after substantive changes. If build/runtime-sensitive code changed, also run `bun run build`

---
> Source: [vtemian/octto](https://github.com/vtemian/octto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
