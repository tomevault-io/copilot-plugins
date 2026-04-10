## blogpoint

> - **Build:** `node ace build`

# AGENTS.md

## Build, Lint, and Test Commands

- **Build:** `node ace build`
- **Dev server:** `node ace serve --hmr`
- **Start:** `node bin/server.js`
- **Lint:** `eslint .`
- **Format:** `prettier --write .`
- **Typecheck:** `tsc --noEmit`
- **Test all:** `node ace test`
- **Test single file:** `node ace test path/to/test_file.ts`

## Code Style Guidelines

- **Imports:** Use ES module syntax and path aliases (e.g., `#models/user`).
- **Formatting:** Enforced by Prettier and ESLint. Use semicolons, double quotes for strings, and smart `eqeqeq`.
- **Types:** Use TypeScript everywhere. Prefer explicit types for functions, models, and variables.
- **Naming:** PascalCase for classes, camelCase for variables/functions, UPPER_CASE for constants.
- **Error Handling:** Use AdonisJS 6 response helpers (e.g., `response.notFound`). Always check for null/undefined.
- **Models:** Use decorators for columns/relations. Use `declare` for class fields.
- **Async:** Use async/await for all asynchronous code.
- **Conventions:** Follow AdonisJS 6 conventions for controllers, models, and middleware.
- **Pre-commit:** Run Prettier and ESLint before committing.

_No Copilot or Cursor rules present._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arafatamim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arafatamim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
