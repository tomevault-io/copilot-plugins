## spark-novel

> This repository is split into two npm projects:

# Repository Guidelines

## Project Structure & Module Organization
This repository is split into two npm projects:

- `client/`: React 19 + TypeScript + Vite UI. Code lives in `client/src`, with pages in `src/pages`, shared UI in `src/components`, state in `src/stores`, utilities in `src/lib`, and assets in `src/assets`.
- `server/`: Fastify + TypeScript API. Bootstrap is in `server/src/server.ts`, routes in `src/routes`, shared helpers in `src/lib`, plugins in `src/plugins`, and Prisma files in `server/prisma`.

Build outputs go to `client/dist` and `server/dist`; do not edit them directly.

## Build, Test, and Development Commands
Run commands from the relevant package directory:

- `cd client && npm run dev`: start the Vite dev server on port 5173 with `/api` proxied to the backend.
- `cd client && npm run build`: run TypeScript project checks and create a production bundle.
- `cd client && npm run lint`: run ESLint for frontend `.ts` and `.tsx` files.
- `cd server && npm run dev`: start the Fastify server with `tsx watch`.
- `cd server && npm run build`: compile the backend to `server/dist`.
- `cd server && npm run db:migrate`: apply Prisma migrations against the configured database.
- `cd server && npm run db:generate`: regenerate Prisma client types after schema changes.

## Coding Style & Naming Conventions
Follow the style already used in each package instead of forcing one global format. In `client/`, use lowercase kebab-case filenames such as `bottom-nav.tsx`; React components stay PascalCase. In `server/`, keep route and helper files lowercase with hyphens or clear nouns such as `error-handler.ts` and `stories.ts`.

Use TypeScript strictness as written, 2-space indentation, and descriptive exports. Frontend linting is enforced with ESLint; backend changes should at least pass `npm run build`.

## Testing Guidelines
Automated tests are not configured yet. Until a test runner is added, treat `client` linting, `server` compilation, and manual API/UI smoke checks as the minimum validation. When adding tests later, place them beside the feature or in a `__tests__` folder and name them `*.test.ts` or `*.test.tsx`.

## Commit & Pull Request Guidelines
Recent git history uses non-descriptive messages (`1`, `2`). Do not continue that pattern; write short, imperative commit subjects such as `feat(client): add materials filter` or `fix(server): validate JWT expiry`.

Pull requests should include a concise summary, affected area (`client` or `server`), setup or migration notes, and screenshots for UI changes. Link the related issue when one exists, and list commands used to verify the work.

---
> Source: [xhongc/spark-novel](https://github.com/xhongc/spark-novel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
