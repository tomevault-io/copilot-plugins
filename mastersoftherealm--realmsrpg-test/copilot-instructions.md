## realms-project

> RealmsRPG project structure and tech stack


# RealmsRPG Project

**Stack:** Next.js (App Router), React, Tailwind, **Supabase** (PostgreSQL, Auth, Storage), **Vercel**. Database: Supabase client + raw SQL; all tables in `public`; no Prisma. **Schema source of truth:** `src/docs/SUPABASE_SCHEMA.md`.

**Key directories:**
- `src/app/` — Routes: `(main)/` (characters, library, codex, creators), `(auth)/` (login, register)
- `src/components/` — `ui/` (primitives), `shared/` (reusable), `character-sheet/`, `creator/`, `codex/`
- `src/hooks/`, `src/stores/`, `src/lib/`, `src/services/`, `src/types/`

**Deploy:** `vercel deploy` or push to connected repo — Vercel only.

**Before PR:** Run `npm run build`. CI runs build + `scripts/reconcile_tasks.js`.

---
> Source: [MastersoftheRealm/RealmsRPG-Test](https://github.com/MastersoftheRealm/RealmsRPG-Test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
