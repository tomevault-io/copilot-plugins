## xxf-ai-server

> This repository is a `pnpm` workspace monorepo. `apps/server` contains the Fastify API gateway, Drizzle migrations, worker entrypoints, and Vitest tests. `apps/web` contains the Next.js 15 App Router frontend for the marketing site, admin area, and user console. `packages/shared/src` holds cross-app TypeScript types and enums. Long-form design, deployment, security, and ADR notes live in `docs/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a `pnpm` workspace monorepo. `apps/server` contains the Fastify API gateway, Drizzle migrations, worker entrypoints, and Vitest tests. `apps/web` contains the Next.js 15 App Router frontend for the marketing site, admin area, and user console. `packages/shared/src` holds cross-app TypeScript types and enums. Long-form design, deployment, security, and ADR notes live in `docs/`.

## Build, Test, and Development Commands
- `pnpm install`: install workspace dependencies.
- `pnpm dev`: start all workspace `dev` scripts in parallel.
- `pnpm build`: build every package and app.
- `pnpm lint`: run workspace lint scripts. Note: server lint is still a placeholder.
- `pnpm typecheck`: run TypeScript checks across the repo.
- `pnpm test`: run workspace tests; currently this is server-side Vitest coverage.
- `pnpm --filter @xxf/server db:migrate`: apply database migrations.
- `pnpm --filter @xxf/server admin:create -- --email admin@example.com --password '...'`: bootstrap an admin account.

## Coding Style & Naming Conventions
Use Node 20+, `pnpm` 9+, and 2-space indentation as defined in `.editorconfig`. Prettier is the formatter of record: single quotes, semicolons, trailing commas, `printWidth` 100. Prefer TypeScript ES modules, small composable modules, and descriptive filenames. Use `page.tsx` for App Router pages, `*.test.ts` for tests, and kebab-case branch names such as `feature/invite-flow`.

## Testing Guidelines
Vitest is configured in `apps/server`. Keep tests close to the code they validate, for example `src/core/accounts/token.test.ts`. Run `pnpm test` before opening a PR; for focused work, use `pnpm --filter @xxf/server test:watch`. There is no dedicated web test suite yet, so at minimum run `pnpm typecheck` and validate affected UI flows locally.

## Commit & Pull Request Guidelines
Follow Conventional Commits, which matches recent history: `feat(web): ...`, `fix(announcements): ...`, `docs(readme): ...`, `security: ...`. Keep each PR scoped to one feature or fix. Include a short description of the problem, the approach, and any tradeoffs. Update docs and `CHANGELOG.md` when behavior changes, and include screenshots for visible web/admin UI changes.

## Security & Configuration Tips
Never commit `.env`, OAuth tokens, or production secrets. Start from `.env.example`, then run Postgres and Redis with `docker compose up -d postgres redis`. For security-sensitive changes, review `docs/security.md` and the relevant ADRs before editing auth, billing, or relay logic.

---
> Source: [xxf66666/xxf-ai-server](https://github.com/xxf66666/xxf-ai-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
