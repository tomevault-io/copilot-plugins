## tanstack-start-cloudflare-workers-starter

> Rule #1: Any important change, constraint, or reminder MUST be added to or removed from `CLAUDE.md` accordingly.

# CLAUDE.md
Rule #1: Any important change, constraint, or reminder MUST be added to or removed from `CLAUDE.md` accordingly.

## Development Workflow
📖 详见 [.agent/workflows/feature-development.md](.agent/workflows/feature-development.md) 或使用 `/feature-development` 命令

## Feature Architecture (Hollow Route)
- **Route files** (`src/routes/`) are **glue code only**: auth checks + component render
- **Feature modules** (`src/features/`) contain all business logic:
  - `components/` — React components with an `index.ts` barrel export
  - `types.ts` — TypeScript interfaces
  - `schemas.ts` — Zod validation (if needed)
  - `api.ts` — Server function re-exports
  - `utils.ts` — Helper functions
- **Current features**: `auth/`, `dashboard/`, `file-upload/`

## Observability

**Cloudflare Native:**
```bash
wrangler tail cloudflare-test-03 --format pretty
```

**Axiom (OpenTelemetry Auto-Instrumented):**
- Config: `src/worker-entry.ts` wraps TanStack Start with `instrument()`

## E2E Testing & Debug
📖 详见 [.agent/workflows/feature-development.md](.agent/workflows/feature-development.md#7-e2e-测试--验证功能)

## Conventions that matter
- **Path aliases**: Use `@/*` for `./src/*` imports
- **Auth schema**: Never edit `src/db/auth.schema.ts` manually — run `pnpm auth:update` after changing auth config
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` plugin
- **Package manager**: pnpm only (enforced by `packageManager` field)
- **Node version**: Requires Node >=18.0.0
- **Testing**: 📖 详见 [devlogs/development-workflow.md](devlogs/development-workflow.md#2-测试)

## Dangerous areas / footguns
- **BETTER_AUTH_SECRET**: Must be set via `wrangler secret put BETTER_AUTH_SECRET` — never put in `wrangler.jsonc` vars
- **GOOGLE_CLIENT_ID / GOOGLE_CLIENT_SECRET**: Must be set via `wrangler secret put` — Google OAuth credentials from Cloud Console
- **PUBLIC_URL**: Must be set in Cloudflare dashboard for production (prevents host header injection)
- **Runtime DB init**: `src/db/init.ts` auto-creates tables on first request — don't rely on this for schema updates in prod; use `pnpm db:migrate:prod`
- **wrangler.jsonc placeholders**: `database_id` and `database_name` need real values for deployment
- **Env vars for drizzle-kit prod**: `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_DATABASE_ID`, `CLOUDFLARE_D1_TOKEN` required in `.env` for `db:migrate:prod`

## Authentication
- **Users**: Google OAuth only (UI shows "Sign in with Google")
- **E2E Tests**: Password login via `/api/auth/sign-in/email` API (hidden from UI)
- Both `socialProviders.google` and `emailAndPassword` enabled in `src/auth/index.ts`

## Notes
- Local D1 state lives in `.wrangler/state/v3/d1/` (auto-discovered by `drizzle.config.ts`)
- `routeTree.gen.ts` is auto-generated — do not edit
- Run `pnpm exec playwright install chromium` on first setup to install browser
- **AI Workflows**: `.agent/workflows/*.md` contains reusable workflow definitions for AI assistants. Use `/feature-development` to invoke the standard feature development flow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darklight2064)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/darklight2064)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
