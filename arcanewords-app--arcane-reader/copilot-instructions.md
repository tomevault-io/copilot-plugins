## arcane-reader

> Core coding standards and project structure


# Core Standards

## Code style

- TypeScript strict. Preact (use `preact`, not `react`).
- Functional components. Named exports.
- 2 spaces, LF. See `.editorconfig`.
- Run `npm run lint` and `npm run typecheck` before committing.

## Structure

- `src/client/` — UI (Preact, Vite)
- `src/engine/` — translation pipeline, glossary, prompts
- `src/services/` — import/export, auth, storage
- `src/storage/` — DB layer
- `src/middleware/` — Express middleware

## Patterns

- Prefer `@file.ts` references over copying code into rules.
- Use existing components from `src/client/components/ui/` (Button, Modal, Input, etc.).
- DB: snake_case columns. See `@src/storage/database.ts` for types.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arcanewords-app) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
