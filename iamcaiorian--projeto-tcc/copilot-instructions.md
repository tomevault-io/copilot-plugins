## projeto-tcc

> Project context and architecture


# Project Context

This repository is part of an academic experiment about prompt engineering for frontend code generation with LLMs.

The frontend architecture has already been defined and must be respected.

## Frontend stack

- React
- TypeScript
- Vite
- TailwindCSS

## Planned architecture

The folder structure is already intentional and should be preserved.

Main structure:

- `src/app` → application bootstrap and routing
- `src/pages` → page-level screens
- `src/features` → domain-specific code
- `src/shared` → shared reusable code
- `src/styles` → global styles and tokens
- `src/assets` → static assets

## Important implementation rule

Existing files were intentionally created for future implementation during the experiment.

If a file already exists, implement code inside that file instead of creating a new one.

The assistant may inspect the entire repository to understand context before writing code.
The assistant should use the full repository as context when needed.

## File ownership by domain

- task-related code should stay inside `src/features/tasks`
- todolist-related code should stay inside `src/features/todolists`
- shared reusable code should stay inside `src/shared`
- page screens should stay inside `src/pages`

Do not reorganize the project structure unless explicitly requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamcaiorian) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
