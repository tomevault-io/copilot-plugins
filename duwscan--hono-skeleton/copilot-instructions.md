## hono-skeleton

> - `src/index.ts`: Hono server entry. Serves `/api`, OpenAPI at `/doc`, Swagger UI at `/ui`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/index.ts`: Hono server entry. Serves `/api`, OpenAPI at `/doc`, Swagger UI at `/ui`.
- `src/app/<feature>/usecase/*`: Use‑case implementations (e.g., `src/app/users/usecase/createUser.ts`).
- `src/app/<feature>/schema.ts`: Zod inputs/validation for that feature.
- `src/app/<feature>/queries.ts`: Drizzle queries; no repository layer.
- `src/routes/*`: Route groups composed into `/api`.
- `src/middleware/*`: Cross‑cutting concerns (auth, sessions, errors).
- `src/core/*`: Shared helpers (errors, result, validation).
- `src/config/*`: Runtime config (auth, database).
- `src/db/*`: Drizzle schema, relations, migrations, and metadata.

## Architecture & Use‑Case Structure
- Organize features under `src/app/<feature>/` with a `usecase/` folder (not dynamic folder names). Example: `src/app/users/usecase/createUser.ts`.
- Keep each use case self‑contained: `schema.ts` (zod input), `usecase/*.ts` (core logic per action), and `queries.ts` for DB access. Do not add a `repo.ts` layer.
- Do not use DTOs. Return plain objects/domain types; map shapes at the route boundary if needed.
- Routes map 1:1 to use cases; keep framework objects at the edge. Convert `Hono` context to plain params in the route layer.
- Prefer small duplication over premature abstractions so use cases stay independent and easy to change.
- Names: verbs for use cases (`createUser`, `listUsers`), nouns for models. Keep private helpers inside the use‑case folder.

## Build, Test, and Development Commands
- `pnpm dev`: Start local dev server with tsx watch.
- `pnpm build`: Type‑check and compile TypeScript to `dist/`.
- `pnpm start`: Run compiled server (`dist/index.js`).
- `pnpm db:change`: Generate a new Drizzle migration from `src/db/schema.ts`.
- `pnpm db:migrate`: Apply pending migrations to the configured DB.
- `pnpm db:pull`: Introspect DB to update schema.
- `pnpm betterauth:generate`: Generate auth schema from `src/config/auth.ts`.

Example: `DB_HOST=localhost DB_PORT=3306 pnpm db:migrate`

## Coding Style & Naming Conventions
- TypeScript, strict mode; ESM with `"type": "module"` and NodeNext. Keep import paths with `.js` extension in TS (matches runtime resolution).
- Indentation: 2 spaces; prefer `camelCase` for variables/functions, `PascalCase` for types/classes, `kebab-case` for filenames.
- Modules: favor named exports for utilities; default export for route groups when convenient.

## Testing Guidelines
- No test framework is configured yet. If adding tests, prefer Vitest.
- Location: co‑locate as `*.test.ts` near sources (e.g., `src/core/result.test.ts`).
- Aim for fast unit tests around `core`, middleware, and route handlers.

## Commit & Pull Request Guidelines
- Commits should be scoped and descriptive. Prefer Conventional Commits (e.g., `feat(routes): add /dummy/hello`).
- PRs must include: summary, rationale, before/after notes, and any schema/migration impact. Link issues when applicable and include screenshots for API docs/UI if relevant.

## Security & Configuration Tips
- Create `.env` with `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD`, `DB_DATABASE`. Do not commit it.
- Run migrations before starting the server. Validate API at `/doc` and `/ui` locally.

## Error Handling
- Controller/route layer: do not add `try/catch`. Throw errors and let the centralized error middleware handle them at runtime (`dist/src/middleware/errors-handler.js` after build; source at `src/middleware`).

---
> Source: [duwscan/hono-skeleton](https://github.com/duwscan/hono-skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
