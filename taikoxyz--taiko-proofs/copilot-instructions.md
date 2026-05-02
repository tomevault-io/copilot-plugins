## taiko-proofs

> - `apps/web`: Next.js 14 UI (App Router in `apps/web/app`, UI pieces in `apps/web/components`, helpers in `apps/web/lib`).

# Repository Guidelines

## Project Structure & Module Organization
- `apps/web`: Next.js 14 UI (App Router in `apps/web/app`, UI pieces in `apps/web/components`, helpers in `apps/web/lib`).
- `apps/api`: NestJS API and indexer (`apps/api/src`), Jest tests in `apps/api/test`, Prisma schema/migrations in `apps/api/prisma`.
- `packages/shared`: Shared TypeScript types exported from `packages/shared/src`.
- `docs`: Operational notes in `docs/runbook.md`; inbox event flow in `docs/event-architecture.md`.
- `ARCHITECTURE.md`: High-level system overview.

## Build, Test, and Development Commands
- `pnpm install`: Install workspace dependencies.
- `pnpm dev`: Run all workspace dev servers.
- `pnpm --filter @taikoproofs/api dev`: Start the API locally.
- `pnpm --filter @taikoproofs/web dev`: Start the web app locally.
- `pnpm --filter @taikoproofs/api exec prisma migrate deploy`: Apply DB migrations.
- `docker compose up -d`: Optional local Postgres (see `docs/runbook.md`).
- `pnpm build`, `pnpm lint`, `pnpm test`: Run workspace builds, linting, and tests.

## Coding Style & Naming Conventions
- TypeScript throughout; use 2-space indentation as seen in existing files.
- ESLint is the baseline (`pnpm -r lint`), with `eslint-config-prettier` in the API.
- NestJS conventions in `apps/api` (`*.module.ts`, `*.service.ts`, `*.controller.ts`).
- React components use PascalCase filenames (e.g., `apps/web/components/BatchesView.tsx`).

## Testing Guidelines
- API tests use Jest + ts-jest; files live in `apps/api/test` and use `*.spec.ts`.
- Run API tests with `pnpm --filter @taikoproofs/api test`.
- Web and shared packages currently have placeholder test scripts; add real tests as needed and update their `test` scripts.

## Commit & Pull Request Guidelines
- No commit history is present yet, so no enforced message convention; keep subjects short and imperative.
- PRs should describe scope, list testing (commands run), and call out env/migration changes.
- Include screenshots or short clips for UI changes in `apps/web`.

## Configuration & Environment
- Copy `apps/api/.env.example` and `apps/web/.env.example` to `.env` before running locally.
- Do not commit secrets; keep `DATABASE_URL`, `RPC_URL`, and public URLs in local or deployment envs.

---
> Source: [taikoxyz/taiko-proofs](https://github.com/taikoxyz/taiko-proofs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
