## proppi

> - `app/` holds Next.js App Router pages, layouts, and API routes (e.g., `app/api/process-image/route.ts`).

# Repository Guidelines

## Project Structure & Module Organization

- `app/` holds Next.js App Router pages, layouts, and API routes (e.g., `app/api/process-image/route.ts`).
- `components/` contains shared UI and feature components; shadcn/ui primitives live in `components/ui/`.
- `lib/` provides data access, actions, and utilities (`lib/db/`, `lib/actions/`, `lib/style-templates.ts`).
- `hooks/` includes React hooks (naming pattern: `use-*.ts` or `use-*.tsx`).
- `emails/` holds React Email templates.
- `public/` stores static assets.
- `trigger/` contains Trigger.dev workflows for background processing.

## Build, Test, and Development Commands

- `pnpm dev` runs the Next.js dev server.
- `pnpm build` builds the production app.
- `pnpm start` serves the production build.
- `pnpm lint` runs ESLint checks.
- `pnpm db:push` syncs Drizzle schema to the database.
- `pnpm db:generate` generates Drizzle migrations.
- `pnpm db:studio` opens Drizzle Studio for local inspection.
- `pnpm email` previews email templates from `emails/`.
- `pnpm trigger` runs Trigger.dev workflows locally.

## Coding Style & Naming Conventions

- TypeScript and React are the default; prefer `.ts`/`.tsx` in `app/`, `components/`, and `lib/`.
- Use existing component patterns and Tailwind CSS utility classes for styling.
- File naming is kebab-case for components and routes (e.g., `project-detail-content.tsx`).
- Lint with `pnpm lint` before sending PRs; follow ESLint guidance in `eslint.config.mjs`.

## Testing Guidelines

- No dedicated test runner is configured yet.
- For changes with risk, include manual verification steps in your PR description (e.g., “Open `app/dashboard/page.tsx` and verify project cards load”).

## Commit & Pull Request Guidelines

- Use Conventional Commits (e.g., `feat: add video wizard step`, `fix: handle upload errors`).
- PRs should include a clear description, linked issues (if any), and screenshots or screen recordings for UI changes.
- Note any schema or env updates (e.g., changes to `drizzle.config.ts` or `.env` keys).

## Configuration & Security Tips

- Copy `.env.example` to `.env.local` and keep secrets out of commits.
- Database access uses `DATABASE_URL`; AI processing uses `FAL_API_KEY`.

---
> Source: [Codehagen/Proppi](https://github.com/Codehagen/Proppi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
