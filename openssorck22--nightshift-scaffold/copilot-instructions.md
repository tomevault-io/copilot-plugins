## nightshift-scaffold

> [One-line description of the product and who it's for.]

# [Project Name] — Agent Conventions

## What This Is
[One-line description of the product and who it's for.]

## Stack
- **Framework**: Next.js 16 (App Router, `src/app/`)
- **Language**: TypeScript (strict mode, zero `any`)
- **Styling**: Tailwind CSS v4
- **Components**: shadcn/ui (`src/components/ui/`). Always use these — never build custom dialogs, selects, etc.
- **Database**: Supabase (Auth + Postgres + RLS)
- **State**: Zustand stores in `src/store/`
- **Icons**: Lucide React (`lucide-react`)
- **Charts**: Recharts with gradient fills

## Commands
- `npm run dev` — start dev server
- `npm run build` — production build
- `npx tsc --noEmit && npx eslint . && npm run build` — pre-push check (zero errors)

## Git Workflow

**For direct sessions with Kevin:**
- Commit directly to `main` and push. No branches, no PRs.
- Vercel auto-deploys from main.

**For squad agents (heartbeat/cron):**
- Branch: `feat/US-XXX-short-description` or `fix/US-XXX-short-description`
- PR: always `--draft`, descriptive title, body with what/why
- One logical change per PR, 1-3 files max
- PRs auto-merge when CI passes + Claude review approves

**Both:**
- Commit messages: concise, imperative ("Add X" not "Added X")
- REQUIRED before first commit: `git config user.email "ssorck22@gmail.com" && git config user.name "Kevin Cross"`
- Run `npx tsc --noEmit` before pushing. Zero TS errors or don't push.

### Before You Start (REQUIRED)
1. `gh pr list --state open` — check for open PRs touching the same files
2. `git log --oneline -10` — check if your change already landed
3. If creating a migration: `ls supabase/migrations/ | tail -5` for next number

## Architecture
```
src/
  app/
    (dashboard)/          -- Protected dashboard pages
    login/                -- Auth pages
    api/                  -- API routes
      auth/callback/      -- OAuth callback
  components/
    ui/                   -- shadcn primitives (DO NOT MODIFY)
    error-boundary.tsx    -- Reusable error boundary
  lib/
    supabase/             -- Server + client factories
    api-auth.ts           -- getUser() helper
    utils.ts              -- cn() helper
  store/                  -- Zustand stores
  types/
    database.ts           -- ALL DB types live here
```

## Coding Standards

### TypeScript
- Zero `any` types. Use `unknown` + type guard if you must.
- All types live in `src/types/database.ts`. Don't guess enum values — check the file.
- When you add columns to a type, update EVERY route that inserts into that table.
- Run `npx tsc --noEmit` after any type change.

### React / Next.js
- Server Components by default. Only add `'use client'` when you need interactivity.
- No setState inside useEffect body. Use key-prop remount pattern or async callback.
- NEVER self-fetch your own API routes from server components. Query Supabase directly instead. (See LESSONS.md)
- shadcn/ui compound components MUST have their Root wrapper. Never use Field.Label standalone. (See LESSONS.md)

### Supabase Patterns
- **org_id on every query** — no exceptions. If you forget, data leaks between tenants.
- **Soft deletes everywhere** — `deleted_at TIMESTAMPTZ`, never hard DELETE.
- **RLS on all tables** — use `get_user_org_id()` function, not subqueries (causes infinite recursion).
- Client-side: `import { createClient } from '@/lib/supabase/client'`
- Server-side: `import { createClient } from '@/lib/supabase/server'`
- Service role (bypass RLS): `import { createServiceClient } from '@/lib/supabase/server'`

### API Routes
```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getUser } from '@/lib/api-auth';

export async function GET(req: NextRequest) {
  const user = await getUser();
  if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  // All queries scoped by user.org_id
  // ...
  return NextResponse.json({ data });
}
```

### Design System
- Use Tailwind classes, never inline styles
- Rounded: `rounded-xl` for cards, `rounded-lg` for buttons/inputs
- Shadows: `shadow-sm`
- Generous spacing — nothing cramped. Let elements breathe.
- Status badges: pill-shaped with soft background colors
- Recharts: gradient fills, not flat colors
- Skeletons for loading states, not spinners

### ESLint Rules That Break CI
1. `prefer-const` — never use `let` if the variable is not reassigned
2. `@typescript-eslint/no-unused-vars` — remove ALL unused imports
3. No `console.log` in `src/` (except error boundaries and instrumentation)
4. No `any` types

## Deploy
- Vercel auto-deploys from `main`
- `vercel.json` ignores non-production deploys
- Set ALL env vars before first deploy: `vercel env ls`
- Git identity must be `ssorck22@gmail.com` — Vercel rejects unknown committers

## What NOT To Do
- Do NOT modify `src/components/ui/` — those are shadcn primitives
- Do NOT use inline styles — Tailwind only
- Do NOT hardcode org IDs, user IDs, or API keys
- Do NOT create files outside the established structure
- Do NOT bundle unrelated changes in one PR — one story = one PR
- Do NOT use `admin.auth.admin.listUsers()` — it fetches ALL users
- Do NOT add eslint-disable directives preemptively — only when confirmed needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openssorck22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
