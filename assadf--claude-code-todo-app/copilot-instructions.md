## claude-code-todo-app

> 69# Claude Code Guidelines

69# Claude Code Guidelines

### Purpose

These rules ensure maintainability, safety, and developer velocity.
**MUST** rules are enforced by CI; **SHOULD** rules are strongly recommended.

## Tech Stack

The following tech **MUST** be used in this application:

- Frontend and Backend: Next.js
  - Styling/UI: Tailwind CSS
  - State Management: Zustand
  - Forms & Validation: React Hook Form (forms), Zod (validation)
  - API & Data Fetching: SWR
  - Authentication: next-auth
  - Utility: classnames
  - ORM: Mongoose
  - Unit/Integration Testing: JEST
  - E2E Testing: Playwright
- Database: MongoDb
- Containerization: Docker and Docker-Compose
- CI/CD: GitHub Actions/Workflows

The following tools/packages **MUST** be used for code quality and formatting:

- Prettier
- ESLint
- TypeScript
- Stylint
- Husky
- lint-staged

## Implementation Best Practices

### Before Coding

- **BP-1 (MUST)** Ask the user clarifying questions.
- **BP-2 (SHOULD)** Draft and confirm an approach for complex work.
- **BP-3 (SHOULD)** If ≥ 2 approaches exist, list clear pros and cons.

---

### While Coding

- **C-1 (MUST)** Follow TDD: scaffold stub -> write failing test -> implement.
- **C-2 (MUST)** Name functions with existing domain vocabulary for consistency.
- **C-3 (MUST)** All secrets to go in a .env (.env.local for local development) Never hardcode secrets.
- **C-4 (MUST)** All configurations to go in a (.env.local for local development). Never hardcode configuration.
- **C-5 (MUST)** Build applications with the appropriate setup that is optimised to run in Docker for local development and also optimised to run in Docker for production environments.
- **C-6 (SHOULD NOT)** Introduce classes when small testable functions suffice.
- **C-7 (SHOULD)** Prefer simple, composable, testable functions.
- **C-8 (MUST)** Use `import type { … }` for type-only imports.
- **C-9 (SHOULD NOT)** Add comments except for critical caveats; rely on self‑explanatory code.
- **C-10 (SHOULD)** Default to `type`; use `interface` only when more readable or interface merging is required.
- **C-11 (SHOULD NOT)** Extract a new function unless it will be reused elsewhere, is the only way to unit-test otherwise untestable logic, or drastically improves readability of an opaque block.

---

### Testing

- **T-1 (MUST)** For a simple function, colocate unit tests in `*.spec.ts` in same directory as source file.
- **T-2 (MUST)** For any API change, add/extend integration tests in `packages/api/test/*.spec.ts`.
- **T-3 (MUST)** ALWAYS separate pure-logic unit tests from DB-touching integration tests.
- **T-4 (SHOULD)** Prefer integration tests over heavy mocking.
- **T-5 (SHOULD)** Unit-test complex algorithms thoroughly.
- **T-6 (SHOULD)** Test the entire structure in one assertion if possible

  ```ts
  expect(result).toBe([value]); // Good

  expect(result).toHaveLength(1); // Bad
  expect(result[0]).toBe(value); // Bad
  ```

---

### Code Organization

- **O-1 (MUST)** Place code in `packages/shared` only if used by ≥ 2 packages.
- **O-2 (MUST)** Follow Clean code architecture
- **O-3 (MUST)** follow a feature folder structure

---

### Tooling Gates

- **G-1 (MUST)** `prettier --check` passes.
- **G-2 (MUST)** `turbo typecheck lint` passes.

## Features and Designs

Refer to the FEATURES.md file which contains a checklist of all the features that are to be implemented in this application and also list features that are to be implemented in the future (for reference).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/assadf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/assadf)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
