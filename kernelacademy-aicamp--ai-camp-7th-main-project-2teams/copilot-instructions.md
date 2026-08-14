## ai-camp-7th-main-project-2teams

> The repository has two applications. `front/` is a Next.js 16 App Router application; pages and API route handlers live in `front/app/`, reusable UI in `front/components/`, hooks and stores in their named directories, and shared logic in `front/lib/`. Unit tests are colocated in `__tests__/` directories, while Playwright specs live under `front/e2e/`. `extension/` contains the Manifest V3 Chrome extension (`background/`, `content/`, and `popup/`) and its tests. Database changes belong in sequentially numbered `supabase/migrations/`. Product and technical decisions are documented in `scripts/`, `docs/specs/`, and `tasks/`.

# Repository Guidelines

## Project Structure & Module Organization

The repository has two applications. `front/` is a Next.js 16 App Router application; pages and API route handlers live in `front/app/`, reusable UI in `front/components/`, hooks and stores in their named directories, and shared logic in `front/lib/`. Unit tests are colocated in `__tests__/` directories, while Playwright specs live under `front/e2e/`. `extension/` contains the Manifest V3 Chrome extension (`background/`, `content/`, and `popup/`) and its tests. Database changes belong in sequentially numbered `supabase/migrations/`. Product and technical decisions are documented in `scripts/`, `docs/specs/`, and `tasks/`.

## Build, Test, and Development Commands

Install dependencies separately in `front/` and `extension/` with `npm ci`.

- `cd front && npm run dev` starts the web app with Turbopack.
- `cd front && npm run build` produces the production Next.js build.
- `cd front && npm run lint` runs the Next.js ESLint rules.
- `cd front && npm test` runs Vitest once; `npm run test:cov` adds coverage.
- `cd front && npm run test:e2e` runs public Playwright flows; authenticated flows use `npm run test:e2e:authed` and require test credentials.
- `cd extension && npm run build` bundles the extension into `extension/dist/`; `npm run lint` and `npm test` validate it.

## Coding Style & Naming Conventions

Use TypeScript for web code, strict compiler settings, two-space indentation, single quotes, and extensionless imports. Components use PascalCase (`BookmarkCard.tsx`); hooks use `useCamelCase`; helpers and stores use camelCase. Prefer the `@/` alias within `front/`. Extension code is ESM JavaScript. Follow the checked-in ESLint configurations; no repository-wide Prettier configuration exists.

## Testing Guidelines

Use Vitest and Testing Library for unit/component tests, naming files `<module>.test.ts(x)`. Playwright scenarios use `*.spec.ts`. Cover normal, boundary, and failure cases; mock OpenAI, Supabase, and Chrome APIs. Pure logic should target 80% coverage. Never let tests make paid AI calls or expose stored content or embeddings.

## Commit & Pull Request Guidelines

Follow Conventional Commits with an optional scope, as in `feat(search): ...`, `fix(api): ...`, or `docs: ...`. Branch from `develop` using `feature/<issue>-<name>`, `fix/<issue>-<name>`, or `docs/<name>`. PRs target `develop`, require at least one approval, and should include a change summary, `Closes #...`, testing steps, and screenshots for UI changes. Rebase on `develop` and ensure lint, tests, build, and E2E checks pass.

## Security & Configuration

Copy `front/.env.example` for local setup and never commit secrets. Do not return embeddings from APIs, persist or log captured page content, or expose Supabase service-role/OpenAI keys to browser code.

---
> Source: [KernelAcademy-AICamp/ai-camp-7th-main-project-2teams](https://github.com/KernelAcademy-AICamp/ai-camp-7th-main-project-2teams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
