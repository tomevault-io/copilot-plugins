## money-tracker

> money-tracker is a personal finance tracker that connects to Gmail to automatically detect and extract financial transactions from emails using AI (xAI). It is a npm workspace monorepo with a React 19 frontend backed by Supabase (Postgres + Edge Functions).

# AGENTS.md

## Project Overview

money-tracker is a personal finance tracker that connects to Gmail to automatically detect and extract financial transactions from emails using AI (xAI). It is a npm workspace monorepo with a React 19 frontend backed by Supabase (Postgres + Edge Functions).

Stack: TypeScript (strict), React 19, React Router v7, TanStack Query, Tailwind CSS v4, Bun runtime, Supabase (Postgres + Edge Functions + Realtime), Deno (Edge Functions), Zod, React Hook Form, Lucide React, Sonner, Recharts.

## Project Structure

```
money-tracker/
├── packages/
│   └── frontend/         # React 19 app (Bun runtime, Vite-like build)
│       └── src/
│           ├── components/   # Shared UI components
│           ├── hooks/        # TanStack Query hooks
│           ├── pages/        # Route-level page components
│           ├── services/     # Supabase client interactions
│           ├── types/        # Types derived from database.types.ts
│           ├── routes/       # React Router route definitions
│           ├── lib/          # Supabase client, query client, form schemas
│           └── utils/
└── supabase/
    ├── functions/        # Deno Edge Functions
    │   └── _shared/      # Shared helpers (auth, cors, notifications, gmail-auth)
    └── migrations/       # Postgres SQL migrations
```

## Build / Dev Commands

```bash
# Dev server
bun dev                         # all packages
bun --filter '*' dev            # same via filter

# Frontend only
cd packages/frontend
bun dev                         # dev server
bun run build                   # production build
bun run lint                    # ESLint

# Database
bun docker:up                   # start Supabase via Docker
bun docker:down                 # stop
bun docker:db:reset             # reset DB (migrations + seeds)
bun docker:db:types             # regenerate TS types from DB schema
bun docker:db:migration:up      # apply pending migrations

# Supabase Edge Functions (deploy via Supabase CLI)
supabase functions deploy <function-name>
supabase functions serve        # local dev
```

## Code Style

### TypeScript

- Strict mode enabled
- Use `import type` for type-only imports
- Never cast to `any`; use `unknown` or proper types
- Never manually declare database types — always derive from `packages/frontend/src/types/database.types.ts`

### React

- Compose smaller components; avoid massive JSX blocks
- Single component per file
- Avoid `useEffect` unless absolutely needed
- Colocate code that changes together
- No emojis in code or UI unless explicitly requested

### Data Fetching (4-layer pattern)

1. **Service layer** (`src/services/`) — Supabase client calls, throw on error
2. **Type layer** (`src/types/`) — types derived from `database.types.ts`
3. **Hook layer** (`src/hooks/`) — TanStack Query hooks (`useQuery` / `useMutation`)
4. **Component layer** — consume hooks, handle loading/error states

Always invalidate queries after mutations. Set appropriate `staleTime`. Never put Supabase queries directly in components.

### Styling

- Tailwind CSS v4 with global CSS file format
- No hardcoded colors — use Tailwind semantic tokens or CSS variables
- Icons: `lucide-react`
- Use built-in Tailwind values; avoid arbitrary values unless necessary

### Supabase Edge Functions

- Runtime: Deno. Use `jsr:` and `npm:` specifiers, never bare specifiers
- Shared utilities go in `supabase/functions/_shared/`, never cross-import between functions
- Use `Deno.serve` — never import `serve` from deno.land/std
- Auth: internal functions use `requireInternalAuth()` with `INTERNAL_FUNCTIONS_SECRET` from vault

### Commit Messages

Conventional Commits: `<type>[scope]: <description>`
Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, `ci`
Use imperative mood, no period at end, max 72 chars.

## Available Skills

Load these skills with the `skill` tool when relevant:

- **commit** — Write conventional commit messages
- **create-migration** — Guidelines for writing Postgres migrations in `supabase/migrations/`
- **create-rls-policies** — Guidelines for writing Postgres RLS policies
- **create-db-functions** — Guidelines for writing Supabase database functions
- **writing-supabase-edge-functions** — Coding rules for Deno Edge Functions
- **postgres-sql-style-guide** — SQL formatting and naming conventions
- **use-realtime** — Supabase Realtime patterns (broadcast over postgres_changes)
- **supabase-safe-mcp** — Restrict Supabase MCP to safe reads only

## Key Domain Logic

- **Gmail OAuth tokens** live in `user_oauth_tokens` table with `is_active` flag
- **Gmail watches** live in `gmail_watches` table; renewed daily by `renew-watches` edge function
- **Token deactivation** is logged permanently in `token_deactivation_log` table
- **`deactivateTokenAndNotify`** in `supabase/functions/_shared/lib/gmail-auth.ts` is the single point that deactivates tokens — always inserts to `token_deactivation_log` first
- **`renew-watches`** edge function is triggered by pg_cron via `public.renew_gmail_watches()` SQL function daily at 02:00 UTC
- **`INTERNAL_FUNCTIONS_SECRET`** is stored in Supabase Vault and used to authenticate internal edge function calls

---
> Source: [FedericoDeniard/money-tracker](https://github.com/FedericoDeniard/money-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
