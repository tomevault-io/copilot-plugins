## investment-idea-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Baburra.io — a backtesting tool for retail investors to evaluate which KOLs' (Key Opinion Leaders) investment opinions are trustworthy and profitable. Users track investment ideas, record predictions with sentiment, and measure accuracy over time via K-line charts and win rate calculations.

## Commands

```bash
npm run dev              # Start dev server (runs clean-dev.js first via predev)
npm run build            # Production build
npm run type-check       # TypeScript checking
npm run lint             # ESLint
npm run lint:fix         # ESLint auto-fix
npm run format           # Prettier format
npm run format:check     # Prettier check

# Unit tests (Vitest + happy-dom)
npm test                 # Run all tests once
npm run test:watch       # Watch mode
npm run test:coverage    # With coverage
# Run a single test file:
npx vitest run src/domain/calculators/price-change.test.ts

# E2E tests (Playwright)
npm run test:e2e         # Headless
npm run test:e2e:ui      # Interactive UI
```

## Architecture

**Stack:** Next.js 16 (App Router) + React 19 + TypeScript + Supabase (PostgreSQL) + Tailwind CSS 4 + shadcn/ui

### Layered Architecture

```
Pages/Components → Hooks (React Query) → API Routes → Repositories → Supabase
```

- **`src/domain/models/`** — TypeScript interfaces for business entities (KOL, Stock, Post, Draft, Bookmark). All domain models use camelCase.
- **`src/domain/calculators/`** — Pure functions for win rate and price change calculations.
- **`src/domain/services/`** — Domain services (AI sentiment analysis via Gemini).
- **`src/infrastructure/repositories/`** — Data access layer. Each repository maps DB rows (snake_case) to domain models (camelCase) and uses `createAdminClient()` to bypass RLS.
- **`src/infrastructure/api/`** — External API clients (Gemini for AI, Tiingo for stock prices).
- **`src/infrastructure/supabase/`** — Three Supabase client types: `client.ts` (browser), `server.ts` (Server Components/API routes), `admin.ts` (bypasses RLS for API operations).
- **`src/app/api/`** — REST API routes. All return `NextResponse.json()`, use repositories for data access.
- **`src/hooks/`** — React Query hooks wrapping API calls. Each resource has hierarchical query keys (e.g., `kolKeys.detail(id)`) and mutations that invalidate relevant queries on success.
- **`src/stores/`** — Zustand for client-only UI state (sidebar, loading).
- **`src/components/ui/`** — shadcn/ui components (New York style, Radix-based).

### Route Structure

- **`src/app/(app)/`** — Protected routes requiring auth (dashboard, kols, stocks, posts, drafts, bookmarks, settings).
- **`src/app/login/`, `src/app/register/`** — Public auth pages.
- All page and API route constants centralized in `src/lib/constants/routes.ts` (`ROUTES` and `API_ROUTES`).

### Adding a New Resource

Follow this pattern: domain model → repository → API route → hook → component/page. See existing resources (kols, stocks, posts, drafts, bookmarks) as templates.

## Internationalization

Uses **next-intl**. Default locale is `zh-TW` (Traditional Chinese), also supports `en`. Translation files in `src/messages/{locale}/`. Locale stored in `NEXT_LOCALE` cookie. Config in `src/i18n/config.ts`.

## Branch Workflow

**IMPORTANT:** At the start of every local session, **sync local main with GitHub main** before doing anything else:

```bash
git checkout main && git pull origin main
```

Then, if the user does not specify which branch to work on, **ask which branch before making any changes.** Run `git branch` to show available branches and confirm with the user.

- `main` — stable production code. Do not commit directly unless the user explicitly says so.
- Feature/rebrand branches (e.g., `rebrand`) — used for isolated work. Always confirm the active branch before editing files.
- When committing, push to the current branch with `-u` flag if it has no upstream yet.
- When creating a new branch, always branch from `main` unless the user says otherwise.

## Development Workflow (OpenSpec)

This project uses **OpenSpec** for specification-driven development. **All non-trivial changes** (new features, significant refactors, multi-file bug fixes) MUST go through the OpenSpec workflow. Trivial changes (typo fixes, single-line config changes) can skip this.

### Workflow

1. **`/opsx:propose <change-name>`** — Create a change with proposal, design, and task checklist in `openspec/changes/<change-name>/`
2. **`/opsx:apply <change-name>`** — Implement tasks from the checklist, marking each complete
3. **`/opsx:archive <change-name>`** — Archive the completed change to `openspec/changes/archive/`

Use **`/opsx:explore <topic>`** for investigation/research without creating artifacts.

### Session Workflow

```
Session start:
  1. Check active OpenSpec changes: ls openspec/changes/
  2. If continuing work → /opsx:apply <change>
  3. If new non-trivial work → /opsx:propose <change>
  4. Only skip OpenSpec for trivial fixes (typos, single-line config)

Session end:
  1. Archive completed changes → /opsx:archive <change>
  2. Update WEB_DEV_PLAN.md phase status (if a phase changed)
  3. Check off completed BACKLOG.md items (if a user story completed)
```

### Rules

- **Propose before coding.** Do not start implementation without a proposed change, unless the user explicitly says to skip it.
- **One change at a time** is preferred. Only work on parallel changes if the user requests it.
- **Tasks drive implementation.** Follow the task checklist in `tasks.md` — do not add scope beyond what's specified without asking.
- **Specs are the contract.** If implementation drifts from the spec, update the spec first, then the code.
- Artifacts live in `openspec/changes/<change-name>/` — do not manually create or edit these files outside of OpenSpec commands.
- Archive completed changes promptly to keep the workspace clean.

### Directory Structure

```
openspec/
├── specs/              # Project-level living specs (data models, API contracts, AI pipeline)
├── changes/
│   ├── <change-name>/  # Active change (proposal.md, design.md, tasks.md)
│   └── archive/        # Completed changes (searchable history)
```

## Documentation Updates

### When to update docs

| Event | Update |
| --- | --- |
| **Phase completed** or status changed | `docs/WEB_DEV_PLAN.md` — update phase status table |
| **User Story completed** | `docs/BACKLOG.md` — check off the story |
| **DB schema changed** | `openspec/specs/data-models.md` — update table/migration list |
| **API endpoint added/changed** | `openspec/specs/api-contracts.md` — update endpoint table |
| **AI pipeline changed** | `openspec/specs/ai-pipeline.md` — update pipeline docs |
| **OpenSpec change completed** | Run `/opsx:archive <change-name>` |

### What NOT to do

- Do NOT update `WEB_DEV_PLAN.md` on every commit — only on phase-level status changes.
- Do NOT duplicate implementation details in `WEB_DEV_PLAN.md` — that belongs in OpenSpec changes or specs.
- `WEB_DEV_PLAN.md` is a **slim roadmap** (~200 lines). Keep it that way.

## Environment Setup

**IMPORTANT:** `.env*` is gitignored, so `.env.local` is **NOT** available in new worktrees or Cloud environments by default. You must set it up manually.

### Local Worktrees

When working in a local worktree (e.g., `.claude/worktrees/<name>/`), copy `.env.local` from the main repo root:

```bash
cp /c/Cursor_Master/investment-idea-monitor/.env.local .env.local
```

### Cloud Environments

In Cloud (remote) environments, `.env.local` does not exist. The preferred setup is:
1. **Pre-configured:** The user sets environment variables in the Claude Code web UI (claude.ai/code → Environment Settings). If these are set, create `.env.local` from them at session start.
2. **Fallback:** If env vars are not pre-configured, ask the user to provide the required values, then create `.env.local` from `.env.example` and fill them in.

### Required Variables

See `.env.example` for the full list. At minimum you need:
- `NEXT_PUBLIC_SUPABASE_URL` + `NEXT_PUBLIC_SUPABASE_ANON_KEY` — Supabase connection
- `SUPABASE_SERVICE_ROLE_KEY` — Server-side Supabase (bypasses RLS)
- `SUPABASE_ACCESS_TOKEN` — Supabase CLI authentication (generate at dashboard.supabase.com/account/tokens)
- `SUPABASE_DB_PASSWORD` — Database password for CLI operations (found in Project Settings → Database)
- `GEMINI_API_KEY` — AI sentiment analysis
- `TIINGO_API_TOKEN` — Stock price data (US equities + crypto)
- `DEV_USER_ID` — Bypass auth in development

### Previewing

When previewing the production build (both Cloud and local), ensure `.env.local` exists with real credentials so the preview can connect to services and produce meaningful results.

## Dev Server (Preview Tool)

The Claude Preview tool uses `.claude/launch.json` to start the dev server. On Windows, `npm` cannot be spawned directly — use `node` with the Next.js binary instead:

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "next-dev",
      "runtimeExecutable": "node",
      "runtimeArgs": ["node_modules/next/dist/bin/next", "dev", "--webpack"],
      "port": 3000,
      "autoPort": true
    }
  ]
}
```

**In worktrees:** `node_modules` is NOT shared. Run `npm install` in the worktree before starting the dev server.

## Supabase CLI

The project is linked to Supabase project `jinxqfsejfrhmvlhrfjj`. CLI requires `SUPABASE_ACCESS_TOKEN` in the environment.

### Command Safety Tiers

**Free to run** (read-only, no confirmation needed):
```bash
supabase migration list -p "$SUPABASE_DB_PASSWORD"   # Show local vs remote migrations
supabase db push --dry-run -p "$SUPABASE_DB_PASSWORD" # Preview pending migrations
supabase gen types typescript --linked --schema public # Generate TypeScript types
supabase inspect db-size                               # Database size stats
supabase inspect db-table-sizes                        # Per-table sizes
```

**Run with user confirmation** (writes to remote DB or creates files):
```bash
supabase db push -p "$SUPABASE_DB_PASSWORD"           # Apply pending migrations
supabase migration new <name>                          # Create new migration file
```

**Never run** (destructive, data loss risk):
- `supabase db reset` — drops and recreates everything
- `supabase migration repair` — modifies migration history
- `supabase db push --include-all` — re-applies all migrations

### Migration + Type Generation Workflow

After creating or modifying a migration:
1. `supabase db push --dry-run -p "$SUPABASE_DB_PASSWORD"` — preview what will be applied
2. `supabase db push -p "$SUPABASE_DB_PASSWORD"` — apply (with user confirmation)
3. `supabase gen types typescript --linked --schema public > src/infrastructure/supabase/database.types.ts` — regenerate types
4. `npm run type-check` — verify types compile

### Worktree Notes

The Supabase project link is stored in `supabase/.temp/project-ref`. In worktrees, re-link if needed:
```bash
export SUPABASE_ACCESS_TOKEN=<token>
supabase link --project-ref jinxqfsejfrhmvlhrfjj -p "$SUPABASE_DB_PASSWORD"
```

## Key Conventions

- Path alias: `@/` maps to `src/`
- Prettier: single quotes, semicolons, 100 char width, trailing commas (es5), tailwindcss plugin
- DB migrations in `supabase/migrations/`, seed data in `supabase/seed.sql` (full) and `supabase/seed-minimal.sql` (categories only)
- Development auth: set `DEV_USER_ID` env var to bypass login
- Test files: `src/**/*.{test,spec}.{ts,tsx}` (Vitest config in `vitest.config.mts`)
- Documentation in `docs/` (ARCHITECTURE.md, API_SPEC.md, DOMAIN_MODELS.md, WEB_DEV_PLAN.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alan8983) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
