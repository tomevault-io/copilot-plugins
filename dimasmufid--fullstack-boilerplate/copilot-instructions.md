## fullstack-boilerplate

> This repository is a Turborepo with app packages in `apps/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Turborepo with app packages in `apps/`.
- `apps/web/`: Next.js frontend (`src/app`, `src/components`, `src/lib`, `src/hooks`, `public/`).
- `apps/api/`: NestJS backend (`src/` for modules/services/controllers, `test/` for e2e tests).
- Database assets:
  - Web: `apps/web/src/db/` and `apps/web/drizzle/`
  - API: `apps/api/src/db/` and `apps/api/drizzle/`
- Root orchestration: `turbo.json`, `pnpm-workspace.yaml`, root `package.json`.

## Build, Test, and Development Commands
Run commands from the repository root unless noted.
- `pnpm dev`: start all apps in development via Turbo.
- `pnpm dev:web`: run only the Next.js app.
- `pnpm dev:api`: run only the NestJS API.
- `pnpm build`: build all apps.
- `pnpm lint`: run lint tasks across apps.

API-specific commands (from root with filter, or in `apps/api`):
- `pnpm --filter api test`: run unit tests (Jest).
- `pnpm --filter api test:e2e`: run end-to-end tests.
- `pnpm --filter api test:cov`: generate coverage report.

## Coding Style & Naming Conventions
- Language: TypeScript across web and API.
- Use existing aliases in web code, e.g. `@/components`, `@/lib`, `@/hooks`.
- Follow ESLint configs: `apps/web/eslint.config.mjs`, `apps/api/eslint.config.mjs`.
- Web naming: React components in `PascalCase.tsx`; hooks as `useX.ts`.
- Keep formatting consistent with surrounding files; run lint before opening a PR.

## Testing Guidelines
- API uses Jest with `*.spec.ts` naming (`apps/api/src`) and e2e config in `apps/api/test/jest-e2e.json`.
- Add/adjust tests for backend behavior changes.
- Web has no configured test runner yet; if adding one, include scripts and conventions in `apps/web/package.json`.

## Commit & Pull Request Guidelines
- Commit style in history is short, lowercase, imperative (e.g., `add auth page`).
- PRs should include:
  - clear summary of behavior changes,
  - verification steps/commands,
  - screenshots for UI updates,
  - env var or migration notes when auth/database changes are involved.

## Security & Configuration Tips
- Use `.env.example` as the baseline; keep secrets in local env files only.
- Never commit credentials or production tokens.
- Document required environment variables and migration steps in PR descriptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dimasmufid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dimasmufid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
