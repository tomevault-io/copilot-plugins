## crm-app

> @README.md for project overview and setup instructions.

# CRM App

@README.md for project overview and setup instructions.
@package.json for available scripts and dependencies.

## Stack

- Next.js 15 (App Router) · TypeScript strict · Prisma 7 + PostgreSQL · Tailwind v4 · Biome v2

## Commands

```bash
npm run build         # Build (zero errors required)
npx tsc --noEmit      # Type check
npx biome check .     # Lint + format
npx prisma validate   # Schema check
npx vitest run        # Tests
```

## Structure

```
src/app/(app)/       → Authenticated pages (dashboard, contacts, companies, deals, activities)
src/app/(auth)/      → Login, register
src/components/ui/   → Shared UI primitives (Button, Input, Label, Dialog, Select, Textarea)
src/components/<entity>/ → Entity-specific components
src/lib/actions/     → Server actions (CRUD operations)
src/lib/auth.ts      → JWT auth (getSession, getCurrentUser)
src/lib/prisma.ts    → Prisma client singleton (uses @prisma/adapter-pg)
src/types/           → Re-exported Prisma types
prisma/schema.prisma → Database schema
```

## Key Rules

- Server Components by default — `"use client"` only when needed
- All DB pages: `export const dynamic = "force-dynamic"` as first line
- Import Prisma client from `@/lib/prisma` only — never instantiate directly
- Imports use `@/` path alias
- Conventional commits: `feat:`, `fix:`, `chore:`
- Biome only — NO ESLint, NO Prettier
- No `any` types — use Prisma-generated types via `@/types`

## Banned

❌ ESLint / Prettier · ❌ `import *` · ❌ inline styles · ❌ `any` type · ❌ direct PrismaClient instantiation · ❌ raw SQL

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/morgan8889) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
