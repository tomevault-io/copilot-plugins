## arbeitssuche

> This guide is for autonomous coding agents working in this repository.

# AGENTS.md

This guide is for autonomous coding agents working in this repository.
It focuses on how to build/test/lint and which coding rules to follow.

## Project Snapshot

- Stack: TypeScript, Electron, React, Vite, Vitest, Playwright
- Package manager: npm (`package-lock.json` is present)
- Module mode: ESM (`"type": "module"`)
- Path alias: `@/* -> src/*`
- Main layers: `src/utils`, `src/models`, `src/plugins`, `src/repositories`, `src/services`, `src/app`, `src/ui`
- Architecture enforcement: `eslint-plugin-unslop` (see `eslint.config.ts` for the full architecture definition)

## Setup

- Install dependencies: `npm install`
- Run app in dev mode: `npm run dev`

## Build, Lint, and Test Commands

### Core Commands

- `verify` - check everything including build:
  - `npm run verify`

## Running a Single Test (Important)

### Vitest (unit/component)

- Single file:
  - `npm test -- src/models/applicant/resolve.test.ts`
- Single test name pattern:
  - `npm test -- -t "fills missing collections"`
- Single file + name filter:
  - `npm test -- src/models/applicant/resolve.test.ts -t "fills missing collections"`

### Vitest Integration

- Full crawler test suite: `npm run test:crawler`
- Single integration file:
  - `npx vitest run --config vitest.integration.config.ts src/plugins/job-site/<name>.integration-test.ts`
- Existing focused crawler tests:
  - `npm run test:crawler:arbeitsagentur`
  - `npm run test:crawler:xing`
  - `npm run test:crawler:zalando`
  - `npm run test:crawler:dm`

### Playwright (E2E + Visual)

- Single E2E spec file:
  - `npx playwright test e2e/tests-flow/<spec>.ts --config=e2e/playwright.electron.config.ts`
- Single visual spec file:
  - `npx playwright test e2e/tests-templates/<spec>.ts --config=e2e/playwright.visual.config.ts`
- Single Playwright test by title:
  - `npx playwright test --config=e2e/playwright.electron.config.ts --grep "test title"`

## Code Style and Conventions

### Formatting

- Run Prettier via `npm run format` for all `*.ts` and `*.tsx`
- Keep formatting tool-driven; avoid manual style-only edits
- Current behavior is default Prettier (double quotes, semicolons)

### TypeScript and Types

- `tsconfig` uses `strict: true`; keep new code strict-safe
- Prefer explicit types for exported/public APIs
- Use `import type` for type-only imports
- Do not use non-null assertions (`!`) in production code
- Do not use type assertions (`as` / angle-bracket casts); resolve types with proper modeling
- Prefer small helpers instead of deep nested conditionals

### Naming Conventions

- `*.ts` and `*.tsx` filenames must be kebab-case
- Tests use one of these suffixes:
  - `.test.ts`
  - `.test.tsx`
  - `.test-suite.ts`
  - `.integration-test.ts`
- Keep domain naming consistent (`Applicant`, `Vacancy`, `JobSearch`, etc.)

### Imports and Module Boundaries

- Under `src/`, parent imports (`../`) are forbidden
- Use `@/...` for cross-module imports via public surfaces
- Use `./...` for local, same-module imports
- Cross-folder imports should go through `index.ts` or `types.ts`
- Tests are stricter:
  - no parent imports (`../`)
  - relative imports should use `index`/`types` surfaces
  - aliased imports should use public surfaces

### Architecture Rules (eslint-plugin-unslop)

Architecture is enforced by `eslint-plugin-unslop` configured in `eslint.config.ts`. Key rules:

- **import-control**: Enforces module-level allow lists for cross-module imports
  - `utils` — shared, no declared imports
  - `models/*` — may import `models/*`
  - `plugins/*` — may import `plugins/*`, `utils`
  - `repositories/*` — may import `repositories/*`, `models`, `models/*`, `utils`
  - `services/*` — may import `services/*`, `plugins/*`, `models`, `models/*`, `repositories/*`, `utils`
  - `app` and `app/*` — may import `utils`, `models`, `models/*`, `plugins/*`, `repositories/*`, `services/*`
  - `ui/components` — shared, may import `ui/hooks`
  - `ui/layout` — may import `ui/hooks`, `ui/components`, `models`
  - `ui/data` — may import `models`
  - `ui/pages/*` — may import `ui/hooks`, `ui/components`, `ui/layout`, `ui/data`, `models`

- **export-control**: Enforces export patterns (currently inert, no contracts declared)

- **no-false-sharing**: Enforces that shared modules have multiple consumers
  - `utils` and `ui/components` must be consumed by at least 2 distinct entities

- **read-friendly-order**: Enforces helper functions are defined after their first use

- UI page groups must stay isolated (`applicant`, `job-search`, `settings`)
- `plugins`, `repositories`, `services`, `app` should import only allowed upstream surfaces
- Shared-code rule: files in configured shared dirs should be consumed by at least 2 entities
- Self-import-depth rule: avoid deep same-module imports

### Complexity and File Size

- Max cyclomatic complexity: `7`
- Max file length: `500` lines (blank lines/comments ignored)
- `src/utils/*.ts` has stricter max length: `80` lines (except tests)
- Split large files into focused helpers instead of raising limits

### React/UI Rules

- Destructure component props in the function signature
- Do not use a generic `props` object when destructuring is possible
- Keep page-specific logic in `src/ui/pages/*`
- Move shared UI logic to `components`, `layout`, `hooks`, or `data`

### Error Handling

- Throw `Error` for invariants and impossible states
- For recoverable external failures (network/LLM/crawl), catch/log context and continue safely
- Normalize unknown errors before logging or surfacing (for example with `formatError`)
- Preserve cancellation behavior with `AbortSignal` in long-running flows

### Testing Conventions

- Use Vitest APIs (`describe`, `it`, `expect`)
- Keep test names behavior-focused and deterministic
- Prefer testing public surfaces (`index.ts`) instead of internals
- Co-locate tests with related source when practical

## Agent Execution Checklist

- Before committing any changes, run: `npm run fix`
- Before finishing, run at least:
  - `npm run verify`
  - `npm test`
- If crawler/integration logic changed, also run: `npm run test:crawler`
- If UI flow, Electron boot, preload, or IPC changed, also run: `npm run test:e2e`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skhoroshavin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/skhoroshavin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
