## garden-weed-planner

> Garden Weed Planner is an Astro 6 SSR app on Cloudflare Workers with React 19 islands, TypeScript, Tailwind CSS 4, and Supabase auth. Product requirements live in @context/foundation/prd.md; deployment notes live in @context/foundation/deploy-plan.md.

# Repository Guidelines

Garden Weed Planner is an Astro 6 SSR app on Cloudflare Workers with React 19 islands, TypeScript, Tailwind CSS 4, and Supabase auth. Product requirements live in @context/foundation/prd.md; deployment notes live in @context/foundation/deploy-plan.md.

## Critical Rules & Commands

- Use Node `22.14.0` from @.nvmrc and npm; CI installs with `npm ci`.
- Run `npx astro sync`, `npm run lint`, `npm run test`, and `npm run build` before handing off changes that touch app code or config; GitHub Actions runs the same gate on `main` PRs.
- Keep `SUPABASE_URL` and `SUPABASE_KEY` server-only. Declare env access through @astro.config.mjs and store local Cloudflare secrets in `.dev.vars`, not committed files.
- Do not add protected pages without updating `PROTECTED_ROUTES` in @src/middleware.ts.
- Skills must not write to `context/archive/`. Archived changes are immutable; if a resolved target path starts with `context/archive/`, abort with: "This change is archived. Open a new change with `/10x-new` instead."

## Project Structure

- `src/pages/` contains Astro pages; auth routes are under `src/pages/auth/`.
- `src/components/` contains Astro and React UI; shadcn/ui components live in `src/components/ui/` per @components.json.
- `src/lib/` holds Supabase, config-status, and shared helpers; keep product logic there before wiring it into pages.
- `context/foundation/` contains durable product, stack, and deployment decisions; do not put change-scoped notes there.

## Development Workflow

Use npm scripts from @package.json for local development, validation, preview, linting, and formatting.

## Coding Style & Conventions

Use the `@/*` import alias from @tsconfig.json for `src` imports. Prefer Astro components for static/layout work and React components only for client interactivity, as shown by auth forms in `src/components/auth/`. Merge conditional Tailwind classes with `cn()` from @src/lib/utils.ts. API handlers should use uppercase method exports and validate request data before calling Supabase.

## Testing & CI

`npm run test` runs the Vitest domain-unit suite for `src/lib/*.test.ts`; keep tests deterministic and free of Supabase secrets or `.dev.vars` content. CI runs Astro sync, lint, test, and build for PRs; deploys to Cloudflare Workers only happen on pushes to `main`.

## Local Git Hooks

Hook behavior lives in @.husky/pre-commit, @.husky/pre-push, and the related scripts in @package.json. If a hook fails, run the named npm script directly to debug it.

## Commits & PRs

Recent history uses Conventional Commit-style prefixes (`feat:`, `docs:`, `chore:`). PRs should mention affected routes/components, list validation commands run, and call out Supabase schema, RLS, or secret changes explicitly.

<!-- BEGIN @przeprogramowani/10x-cli -->

## 10xDevs AI Toolkit - Module 2, Lesson 3

Review AI-generated code before merge with the implementation review chain:

```
/10x-implement -> /10x-impl-review -> triage -> (/10x-lesson | fix | skip | disagree)
```

Use `/10x-impl-review <change-id>` for implemented-code review before merge. Use `/10x-lesson` when a finding reveals a recurring project rule or agent failure pattern worth recording in @context/foundation/lessons.md.

<!-- END @przeprogramowani/10x-cli -->

---
> Source: [simon2k/garden-weed-planner](https://github.com/simon2k/garden-weed-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
