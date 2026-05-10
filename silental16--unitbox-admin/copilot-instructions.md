## unitbox-admin

> Internal admin dashboard for Unitbox company operations (codename: **unitbox-hq**). NOT the product catalog — that's in `../catalog/`.

# Unitbox HQ — Internal Back-Office

## What is this project

Internal admin dashboard for Unitbox company operations (codename: **unitbox-hq**). NOT the product catalog — that's in `../catalog/`.

Current modules:
- **Bali Developer Pipeline** — BD pipeline with 97 developers for outreach (research, ICP scoring, contact tracking)
- **Preview** — component showcase / design system preview

Planned: accounting, analytics, internal statistics.

## Tech Stack

- **Framework:** Next.js 16 (App Router, Turbopack)
- **UI:** shadcn/ui + Tailwind CSS v4 + Radix UI
- **Database:** Supabase (PostgreSQL)
- **Auth:** Supabase Auth (email/password)
- **Hosting:** Vercel
- **Charts:** Recharts

## Project Structure

```
app/
  (dashboard)/
    developers/    — BD Pipeline page (main feature)
    preview/       — Component showcase
    layout.tsx     — Dashboard shell with sidebar
  auth/callback/   — Supabase auth callback
  login/           — Login page
components/
  developers/      — Developer table, filters, detail sheet
  ui/              — shadcn components
  cards/           — Dashboard card components
  blocks/          — UI block demos
lib/
  data/            — Data fetching, queries, ICP scoring
  supabase/        — Supabase client (browser, server, middleware)
```

## Key Concepts

- **ICP Score** — Ideal Customer Profile score (0-100) for developer prioritization
- **Origin tags** — RU, EU, INT — developer origin/market
- **Projects ratio** — `filled/total` projects per developer

## Commands

```bash
npm run dev    # Start dev server on port 3005
npm run build  # Production build
npm run lint   # ESLint
```

## Environment

`.env.local` must contain:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

## UI Rules

- **ONLY use shadcn/ui components.** No custom UI — everything must be built from shadcn primitives.
- If something can't be done with shadcn — ask the user first before writing custom code.
- Always use `/shadcn-design` skill patterns for composition (cards, tables, forms, dashboards).
- Always use `/shadcn` skill when adding or debugging components.
- Radix UI is OK only as the foundation under shadcn (it's already there). Don't use raw Radix directly.

## Important: Do NOT confuse with

- `../catalog/` — Unitbox product catalog (NestJS + React + Mantine, PostgreSQL in Docker). That's where AI training fills real estate projects.
- This project is the **company internal tool**, not the customer-facing product.

---
> Source: [Silental16/unitbox-admin](https://github.com/Silental16/unitbox-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
