## vehicle-track

> Vehicle Track follows the Next.js App Router plan documented in `documentation/architecture.md`: feature pages and layouts live in `app/`, while HTTP entry points sit in `app/api/*/route.ts` with Zod validation and role guards. Keep business logic inside `lib/services/*` and share typed utilities through `types/`. Supabase schemas and Drizzle migrations belong under `drizzle/`, and reusable Tailwind/shadcn UI should stay in `components/`. Planning artifacts remain in `documentation/` for reference.

# Repository Guidelines

## Project Structure & Module Organization
Vehicle Track follows the Next.js App Router plan documented in `documentation/architecture.md`: feature pages and layouts live in `app/`, while HTTP entry points sit in `app/api/*/route.ts` with Zod validation and role guards. Keep business logic inside `lib/services/*` and share typed utilities through `types/`. Supabase schemas and Drizzle migrations belong under `drizzle/`, and reusable Tailwind/shadcn UI should stay in `components/`. Planning artifacts remain in `documentation/` for reference.

## Build, Test, and Development Commands
- `pnpm dev` — start the App Router dev server for local work.
- `pnpm lint` — run the strict ESLint config required by the architecture gates.
- `pnpm typecheck` — execute `tsc --noEmit` so CI matches local guarantees.
- `pnpm test` — run the smoke/unit suites outlined in `documentation/testing.md`.
- `pnpm build` — produce the optimized Next.js output consumed by Vercel before promoting to prod.

## Coding Style & Naming Conventions
Use TypeScript `strict` mode, avoid `any`, and keep server components the default. Client modules must begin with `"use client"` and remain small wrappers around shared services. Routes stay thin: parse, validate, authorize, then delegate to `lib/services`. Favor PascalCase React components and kebab-case filenames elsewhere. Tailwind + shadcn utilities should create composable pieces over large monoliths. Run Prettier (if configured) after ESLint to keep formatting consistent.

## Testing Guidelines
Follow the smoke-first approach in `documentation/testing.md`: prioritize login, role enforcement, and the primary vehicle CRUD paths. Keep frontend tests minimal yet meaningful—one happy-path per page is acceptable initially—while API/service suites cover error handling and authorization. Add at least one role-specific assertion per describe block. Use Supabase test credentials via local env files so production data stays untouched.

## Commit & Pull Request Guidelines
Existing history shows imperative commits (e.g., “Implement initial project structure…”); continue that tense with a concise scope prefix when possible. Each PR should link the relevant requirement or architecture section, summarize functional changes, and attach screenshots for UI updates. Always include the verification line you ran locally—`pnpm lint && pnpm typecheck && pnpm test`—before requesting review, and wait for approval from a teammate owning the affected module.

## Security & Configuration Tips
Never commit `.env*` files; Vercel and Supabase manage secrets per `documentation/deployment.md`. Middleware must guard `/dashboard/*`, and Better Auth role checks belong at both route and service boundaries. When logging, redact VINs and user emails to honor the privacy constraints called out in the requirements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tonglam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tonglam)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
