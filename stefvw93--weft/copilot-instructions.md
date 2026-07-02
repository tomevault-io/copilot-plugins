## weft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`Weft` is a pnpm monorepo (`weft-workspace`) implementing an Effect-based UI library with strict TypeScript configuration and modern tooling.

Workspace layout (see `pnpm-workspace.yaml`):

- `packages/*` — published library packages:
  - `@weftui/base` (`packages/base`) — shared primitives
  - `@weftui/core` (`packages/core`) — core combinators, sources, streams, boundaries
  - `@weftui/dom` (`packages/dom`) — DOM renderer with `./client` and `./server` entry points
- `examples/*` — standalone runnable example apps, each its own workspace package

## Requirements

- Node.js

See versions in package.json > engines. Package management and all tooling is handled by `vp` (Vite+).

## Development Commands

All commands use the `vp` CLI (Vite+). Run `vp help` for a full list.

### The `pack` step (read this first)

This is a monorepo: `@weftui/dom` and the `examples/*` consume `@weftui/core`/`@weftui/base` as workspace packages, resolved through their **built `dist/`**. Cross-package type-checking is therefore only correct once those packages have been packed.

**Rule: run validation through the `vp run <task>` tasks, never the bare `vp <command>`.** The tasks are declared in the root `vite.config.ts` under `run.tasks` and each one declares `dependsOn: ["pack"]`, so `vp run` always rebuilds the packages first:

- ✅ `vp run check`, `vp run test`, `vp run test:browser` — pack first, then run. Always correct.
- ❌ `vp check`, `vp test` directly — skip `pack`, so against stale/missing `dist/` they report **false** cross-package errors (e.g. spurious `implicit any` from unresolved `@weftui/*` types). Only safe right after a pack.

Treat the task list in `vite.config.ts` (`run.tasks`) as the source of truth for how to validate — if a task exists there, invoke it via `vp run <task>`. Current tasks: `dev`, `pack`, `check`, `test`, `test:browser`.

### Building

```bash
vp build
```

Uses tsdown for fast TypeScript bundling.

### Testing

```bash
vp run test            # Pack, then run all node/jsdom tests
vp run test:browser    # Pack, then run real-browser e2e tests (Playwright)
vp test --watch        # Watch mode (only safe after a pack)
```

Uses Vitest (via Vite+). Node test files follow the pattern `**/*.test.{ts,tsx}`; `*.browser.test.{ts,tsx}` are excluded from `vp run test` and run via `vp run test:browser` (see the `pack` step rule above).

### Checking (format + lint + typecheck)

```bash
vp run check       # Pack, then format, lint, and type-check all files
vp check --fix     # Auto-fix formatting/lint (only safe after a pack)
```

**Important:** Validate via `vp run check` (it packs first — see the `pack` step rule). Use `vp check --fix` for auto-fixing, but only when packages are already built, otherwise it reports false cross-package type errors. Always prefer these over individual lint/format commands.

## Architecture

### TypeScript Configuration

Strict TypeScript setup with:

- `noUncheckedIndexedAccess: true` - Array/object access returns possibly undefined
- `noImplicitReturns: true` - All code paths must return
- `strict: true` - All strict type-checking enabled
- `verbatimModuleSyntax: true` - Import/export syntax preserved
- `isolatedModules: true` - Each file must be transpilable independently
- `noUncheckedSideEffectImports: true` - Side-effect imports must be explicit

Path aliases (configured per package in `packages/*/tsconfig.json`, which extend `tsconfig.base.json`):

- `~/*` maps to that package's `./src/*`

### Code Style

**Toolchain:** This project uses Oxlint (linting) and Oxfmt (formatting) via Vite+, NOT ESLint or Biome.

Oxfmt enforces:

- Tab indentation
- Double quotes for strings

When ignoring lint rules, use Oxlint syntax:

- ✅ Correct: `// oxlint-disable-next-line <rule-name>`
- ❌ Wrong: `// eslint-disable-next-line` or `// biome-ignore`

### Project Structure

- `packages/*/src/` - Source TypeScript files for each library package
- `packages/*/dist/` - Build output (excluded from TypeScript compilation)
- `examples/*/` - Standalone runnable example apps, each its own workspace package with an `app.ts` entry point and `vite.config.ts`
- `docs/` - Documentation
- `plans/` - Design plans and specs
- ES modules only (`"type": "module"` in package.json)

### Examples

The `examples/` folder contains standalone workspace packages demonstrating specific patterns or features (e.g. `keyed-list`, `form-handling`, `ssr-hydration`).

**Rules for examples:**

- Every example must have a co-located README named `readme.md`
- Each example is a self-contained, runnable workspace package (depends on `@weftui/*` via `workspace:*`)
- Include a JSDoc header comment in `app.ts` explaining the example's purpose
- READMEs should include: Overview, Problem, Solution, How It Works, and When to Use sections
- Each example is split into a **side-effect-free `app.ts`** (or `src/app.ts`) that
  `export`s `App` — no top-level `mount`/`hydrate` call — and a thin entry
  (`main.ts`, or `entry-client.ts` for SSR examples) that mounts it and is the file
  referenced by `index.html`. This keeps `app.ts` importable by tests.
- Every example **must include at least one co-located `*.browser.test.ts`** that
  imports `App`, mounts it in a real browser, and asserts the example's headline
  behaviour. Browser tests use `vite-plus/test` globals (never `vitest` directly)
  and run via `vp run test:browser`. See `e2e/specs.md` for conventions and known
  pitfalls (post-mount render tick, ref observers, missing example CSS).

## Coding Standards

### Architecture & Patterns

- Use a hybrid approach combining functional and object-oriented programming
- Effect (effect.website) is the core library - use its patterns throughout
- Prefer Effect's error handling over try/catch (except when it significantly hurts ergonomics)
- Use Services and Layers for dependency injection
- Prefer `pipe(effect, ...)` over `effect.pipe(...)`
- **No JSX.** Weft does **not** use JSX, even though its node descriptors
  (`{ type, props }`) resemble React elements. There is no JSX runtime (no `jsx`
  in any tsconfig) and no `h(Component)` overload — `h.*` only builds string-tag
  and `FRAGMENT` nodes, and components are plain functions that are **called**
  (e.g. `App()`), placing their resulting node in the tree directly rather than
  deferring construction. Do not assume `<Component/>`-style deferred descriptors
  exist or write code that depends on them.

### TypeScript Standards

- Type assertions (`as`, `!`) only when we're "smarter" than the compiler
- `any` is allowed for generic type params and library interop only
- Use explicit type guards over implicit checks
- Prefer generic constraints over flexibility
- Treat data structures as immutable - use `readonly` extensively
- Prefer `Option` > `undefined` > `null` for optional values
- All checks should be type-level when possible
- Use Schema for validation of unknowns and I/O

### Naming Conventions

- Files: kebab-case (e.g., `user-service.ts`)
- Variables/functions: camelCase, with `is*`, `has*`, `should*` prefixes for booleans
- Types/Interfaces: PascalCase, no `I` prefix for interfaces
- Constants (shared): UPPER_SNAKE_CASE
- Prefer named exports; default exports only if absolutely necessary

### Documentation

- All exported functions, types, and values must have JSDoc comments
- JSDoc `@type` annotations can be omitted (TypeScript handles types)
- Include text descriptions for parameters when not self-explanatory
- Inline comments only when needed - avoid commenting obvious code
- TODOs and FIXMEs are acceptable
- Effect Schemas should include descriptions/annotations when not self-explanatory

### Testing

- Follow Test-Driven Development workflow: spec → mock → test → implement
- Co-locate test files (`*.test.ts`) next to source code
- `__tests__/` directory allowed for compound/integration tests and shared fixtures/helpers
- `__type-tests__/` directory for compile-time type tests (see Type Tests section below)
- Write thorough tests against the API surface and specifications in co-located `specs.md` files
- Test naming conventions:
  - Use `describe` for test grouping, `it` or `test` for individual test cases
  - Test case names should match or reference acceptance criteria from specs.md
- Coverage requirements:
  - All acceptance criteria from specs.md must be covered
  - Cover both happy paths and error paths
  - Test all possible error types defined in the Effect error union (expected errors)
  - Include edge cases defined in specifications
- Use Effect testing utilities for testing Effect code
- Real-browser end-to-end tests live in `*.browser.test.{ts,tsx}` files, run via
  `vp run test:browser` (Vitest browser mode + Playwright), and are excluded from
  the default `vp test` run. Every `examples/*` app must have one — see the
  Examples section above and `e2e/specs.md`.

### Type Tests

Type tests verify compile-time behavior for complex type-level features. They use `@ts-expect-error` comments to assert that certain code should NOT compile.

**Location:** `src/**/__type-tests__/*.test-d.ts`

**Running type tests:**

```bash
vp run check
```

**Rules:**

- Type test files use the `.test-d.ts` extension (convention from `tsd` and similar tools)
- Use `@ts-expect-error` to assert code that should fail to compile
- Type tests are type-checked as part of `vp run check`: both `packages/core` and `packages/dom` include `src` (and therefore `src/**/__type-tests__`) in their tsconfig, so the `@ts-expect-error` assertions are enforced by the main typecheck
- Each type test file should be self-contained and test a specific feature

**Example pattern:**

```typescript
// Should compile - valid usage
const _valid: SomeType = validValue;

// @ts-expect-error - Should NOT compile - invalid usage
const _invalid: SomeType = invalidValue;
```

### Specification Files

- Every new feature must have a co-located `specs.md` file (e.g., `dom/feature.ts`, `dom/feature.test.ts`, `dom/feature.specs.md`)
- Existing features without specs should get them retroactively when modified
- Every planning session must start with extensive specification discussion:
  - Ask questions to understand requirements, edge cases, and constraints
  - Draft specifications interactively with the user
  - Iterate on the spec until complete before writing implementation code
- Use "mock first, implement later" approach:
  - Before implementation, create comprehensive mocks using TypeScript's type system and `declare` keyword
  - Define complete API surface: classes/methods, function signatures, constants/variables, exports, type definitions
  - Review mocks to ensure they match specifications and types are complete
- Implementation rules:
  - Only begin actual implementation after mocks and tests are complete
  - Replace type-system level mocks (e.g., `declare` statements) with real code
  - Ensure implementation matches mock signatures exactly
  - Ensure implementation matches co-located specs.md files
  - If implementation reveals mocks/specs need changes: pause implementation and update specs/mocks first (maintain strict spec → mock → test → implement cycle)
  - After implementation: auto-fix with `vp check --fix`, then validate with `vp run check` and `vp run test` (which pack first)
- Specs MUST include:
  - Feature overview and purpose
  - Detailed acceptance criteria
- Specs COULD include:
  - Technical requirements and constraints
  - Dependencies and integrations
  - Expected behavior and edge cases
- Follow a common structure with standard headings, but allow flexibility between specs

### Error Handling

- Use Effect's tagged errors as the primary error handling mechanism
- Error messages should be descriptive and include context/debugging info when useful
- Input validation required only for unsafe input (user input, `unknown` input)
- Handle errors at program edges when possible

### Module Organization

- Organize code by domain, within the relevant workspace package
- Barrel exports (`index.ts`) only for grouping application domains, e.g. in `@weftui/dom`:
  - `src/index.ts` - package root export
  - `src/client/index.ts` - client-side DOM renderer (`@weftui/dom/client`)
  - `src/server/index.ts` - server-side rendering (`@weftui/dom/server`)
- Avoid circular dependencies
- Use `/utils` only for common code that doesn't fit a specific domain

### Effect-Specific Patterns

- Prefer Effect logic throughout the codebase
- Use Effect Schema for data structures and validation
- Wrap functionality in Services when capabilities need to be shared across modules/components
- Manage runtimes only when explicitly required
- `Effect.gen` vs `pipe` depends on the specific feature and readability

### Code Reuse

- Wait for multiple use cases before abstracting - avoid premature abstraction
- Organize shared utilities by domain; use `/utils` only for cross-cutting concerns
- Duplication vs abstraction is case-by-case - prefer duplication over poor abstraction

### Performance

- Readability first, performance second
- Use memoization only when explicitly specified or instructed
- Be mindful of bundle size: import specific items, not `import * as X`
- `Effect.gen` vs `pipe` choice depends on the feature and readability

### Imports

- Use specific imports, avoid `import * as X`

## Meta Rules

- Always discuss new rules and rule changes in Q&A style. Ask a question and await the answer before asking the next question, until sufficient information is provided.

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, and it invokes Vite through `vp dev` and `vp build`. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

Docs are local at `node_modules/vite-plus/docs` or online at https://viteplus.dev/guide/.

## Review Checklist

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to format, lint, type check and test changes.
- [ ] Check if there are `vite.config.ts` tasks or `package.json` scripts necessary for validation, run via `vp run <script>`.
- [ ] If setup, runtime, or package-manager behavior looks wrong, run `vp env doctor` and include its output when asking for help.

<!--VITE PLUS END-->

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:

- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

---
> Source: [stefvw93/weft](https://github.com/stefvw93/weft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
