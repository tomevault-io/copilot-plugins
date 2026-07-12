## suiseki

> This file provides guidance to LLM agents when working with code in this repository.

# AGENTS.md

This file provides guidance to LLM agents when working with code in this repository.

## Project Overview

`suiseki` is a read-only Bun/TypeScript CLI for rendering diffs and code in the terminal.

The product starts as a unified-view `git diff` renderer and grows into a broader terminal code viewer with `view` and `tree` subcommands. It combines Pierre's diff parsing model with Shiki syntax tokenization, then emits ANSI directly so syntax foreground colors and diff background colors can coexist cleanly.

Position `suiseki` as a friendly terminal surface for Pierre's renderer-agnostic packages and Shiki's syntax/theme ecosystem: `@pierre/diffs` first, `@pierre/trees` next, and Shiki throughout. It is a homage and companion, not a fork or replacement.

Core constraints:
- Act as a Unix filter: read input, write ANSI to stdout, exit.
- Never modify user files.
- Compile to a single local `suiseki` binary with Bun.
- Stay in the TypeScript/Bun ecosystem; do not introduce Go tooling.
- Do not build interactive terminal UI behavior unless the project's direction explicitly changes.
- Keep normal `stdout` clean for rendered output. Send errors and diagnostics to `stderr`, and do not introduce service-style logging dependencies such as `pino` unless there is a concrete need.

## Essential Commands

All commands are available via `make`. Run `make` or `make help` to see the full list.

### Development
- `make run` - Run as TypeScript
- `make build` - Build `./bin/suiseki` with `bun build --compile`
- `make start` - Run `./bin/suiseki`
- `make clean` - Remove compiled `bin/` and release `dist/` artifacts

### Testing
- `make test` - Run all tests with coverage

### Code Quality
- `make check` - Run TypeScript type check and Biome linting/formatting
- `make format` - Format code with Biome
- `make check-ci` - CI version of checks (no auto-fix)

## Architecture

### Tech Stack
- **Runtime**: Bun, TypeScript
- **Binary output**: `bun build --compile`
- **Syntax highlighting**: Shiki tokenization and Shiki-compatible themes
- **Diff parsing**: `@pierre/diffs` parsing/iteration utilities
- **ANSI output**: direct ANSI emission, with `ansis` available for helpers
- **Config**: `smol-toml` for TOML config parsing
- **Validation**: Arktype (TypeScript-first runtime validation)
- **Testing**: Bun test
- **Tooling**: Biome (formatting/linting)

## Development Patterns

### Code Consistency
- **Study existing patterns first** - Review similar code to maintain consistency
- **When renaming files, use `git mv`** - Preserves git history

### Git Guidelines
- **Never use `git -C`** — always assume you're in the correct working directory
- Don't mention that code was generated or co-authored by an LLM agent
- Before committing, run `bun check` to apply formatting
- When formatting is the only change, use commit message "fmt"


##### Validator + type naming convention
Define Arktype schemas as `vFoo` and derive named TypeScript types as `Foo` directly below them:
```typescript
export const vSuisekiConfig = type({
	theme: "string",
	view: "'unified' | 'split'",
})
export type SuisekiConfig = typeof vSuisekiConfig.infer
```

##### Runtime validation boundaries
Use Arktype for data crossing runtime boundaries: config files, environment variables, CLI options, theme metadata, and parsed external JSON. Avoid `as` casts at those boundaries; parse and validate first.

##### Shared validators
When reusable validators become necessary, keep them in `src/common/validators.ts` and prefer shared schemas over duplicated inline string validators.

### Testing Guidelines

#### Test File Naming & Imports
- `*.test.ts` - Unit tests (fast, no external dependencies)
- `*.integration.test.ts` - Integration tests that exercise the compiled CLI, filesystem fixtures, or git subprocess behavior
- Both use `import { test, expect, describe } from "bun:test"`

#### What NOT to Test
- Do not snapshot entire ANSI outputs when a smaller fixture or focused assertion can cover the behavior.
- Do not test Shiki, Pierre, Biome, Bun, or Arktype internals.
- Do not add tests for planned features before implementation exists.
- Do not rely on global git/user configuration in tests.

#### What TO Test
- Diff rendering behavior, including file headers, hunk headers, gutters, signs, and ANSI reset correctness.
- Config resolution and Arktype validation.
- CLI input selection: stdin versus git subprocess arguments.
- Edge-case fixtures for binary files, renames, empty diffs, large diffs, and merge conflicts as those features land.

#### Test Structure
- **Describe block per method/function** — group related tests under `describe("methodName", () => { ... })`
- **Classes** — use the class name as the top-level describe, with nested describes per method:
  ```typescript
  describe("ConfigLoader", () => {
    describe("load", () => { ... })
    describe("resolve", () => { ... })
  })
  ```
- **Files with multiple exported functions** — use the filename as the top-level describe so you can run one describe to cover the entire file:
  ```typescript
  // ansi.test.ts
  describe("ansi.ts", () => {
    describe("emitToken", () => { ... })
    describe("emitLine", () => { ... })
    describe("emitReset", () => { ... })
  })
  ```
- **Files with a single exported function** — a single describe with the function name is sufficient (no filename wrapper needed)

#### Test Assertions
- **No conditional branches in tests** — never use `if`, `switch`, or ternaries in test bodies. Every code path must execute unconditionally. Conditional `expect()` calls are a silent-pass bug: if the condition is false, assertions are skipped and the test passes when it shouldn't.
- **No try/catch in tests** — never use try/catch blocks to assert on thrown errors. Use `expect(fn).toThrow()` for sync and `expect(promise).rejects` for async:
  ```typescript
  // wrong: try/catch with assertions inside
  try {
    loadConfig(...)
    assert(false, "config loading should have thrown")
  } catch (error) {
    assert(error instanceof Error, "should be Error")
    expect(error.message).toEqual("Invalid config")
  }

  // right: use rejects.toMatchObject for async throws
  await expect(loadConfig(...)).rejects.toMatchObject({
    message: "Invalid config",
  })

  // right: use toThrow for sync throws (only supports message/instanceof, not property matching)
  expect(() => parseCliOptions(...)).toThrow("Unknown option")
  ```

- Use `assert()` from `node:assert` for type narrowing instead of `if` checks. `assert()` throws immediately on failure, guaranteeing the test fails loudly.
  ```typescript
  // wrong: conditional expect is a silent-pass risk
  if (!result.ok) {
    expect(result.status).toEqual(404)
  }

  // right: assert narrows the type AND fails the test if condition doesn't hold
  assert(!result.ok, "result should not be ok")
  expect(result.status).toEqual(404)
  ```
- **Descriptive variable names in tests** — avoid generic names like `result`. Name variables after what they represent: `renderedDiff`, `parsedConfig`, `gutter`, `patch`, etc. This makes test intent clearer and assertions self-documenting.
- Use `expect().toEqual()` for value comparisons (avoid `toBe()` for consistency)
- Avoid mocking frameworks unless absolutely necessary. Prefer fixtures and small temporary repositories for git behavior.

## Verification After Changes
When changing tests or implementation, always verify:
1. `bun check` - Type check and lint
2. `bun test` - Run all tests with coverage

## Code Style
- Biome handles all formatting and linting
- Kebab-case file naming enforced
- `.types.ts` suffix for type-only files

#### Naming Guidelines
- **No abbreviations in variable/function names** - Use full words: `organization` not `org`, `createOrganization` not `createOrg`. Keep naming consistent with the rest of the codebase.

#### Function Parameter Guidelines
Use parameter objects instead of individual parameters when:
1. **Multiple arguments of same/similar types** - Avoid parameter ordering issues
2. **Three or more arguments** - Always use parameter objects for 3+ args

Parameter object type naming: suffix with `Params` (e.g., `MyFunctionParams`)

```typescript
// ❌ BAD
function renderLine(content: string, foreground: string, background?: string)

// ✅ GOOD
type RenderLineParams = {
	content: string
	foreground: string
	background?: string
}

function renderLine(params: RenderLineParams)
```

## Comment Guidelines

- Don't yell in comments by writing in UPPER CASE
- Avoid redundant comments that restate code
- Add comments for complexity, non-obvious behavior, and "why" explanations
- Comments should explain WHY, not WHAT - code should be self-documenting for WHAT

## Runtime: bun

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build src/cli.ts --compile --outfile bin/suiseki` for the local binary
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads .env, so don't use dotenv.

### Runtime APIs

- Prefer `Bun.file` for straightforward file reads.
- Use `Bun.spawn` for invoking `git diff` and other subprocesses.
- Keep CLI commands read-only unless the user explicitly requests otherwise.
- Do not introduce web servers, React frontends, Redis clients, or database layers for this project.

### Testing

Use `bun test` to run tests.

```ts#index.test.ts
import { test, expect } from "bun:test";

test("hello world", () => {
  expect(1).toBe(1);
});
```

---
> Source: [michaeldlfx/suiseki](https://github.com/michaeldlfx/suiseki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
