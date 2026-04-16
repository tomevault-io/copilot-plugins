## kolata

> Read `docs/PROJECT_OVERVIEW.md` for full technical and business context before making changes.

# Kolata - Car Maintenance Tracker

@AGENTS.md

## Overview
Read `docs/PROJECT_OVERVIEW.md` for full technical and business context before making changes.

## Quick Reference

- **Stack:** Next.js 16 + TypeScript + Tailwind v4 + Supabase + next-intl
- **Package manager:** pnpm
- **Locales:** bg (default), en — both `src/messages/bg.json` and `en.json` must stay in sync
- **DB types:** `src/types/database.ts` — must include `Relationships` array per table
- **shadcn/ui:** Uses `@base-ui/react` (NOT Radix). `SelectValue` needs explicit children for display text.

## Commands
```
pnpm dev       # Dev server
pnpm build     # Production build (must pass before committing)
pnpm lint      # ESLint
```

## Coding Standards
- TypeScript strict, no `any`
- All text visible to users must go through i18n (`useTranslations` / `getTranslations`)
- Add translation keys to BOTH `bg.json` and `en.json` when creating new UI text
- Use `font-semibold` (600) not `font-medium` (500) — project-wide convention
- Base font is `text-lg` (18px) set on `<html>`
- Links wrapping block content need `className="block"` for `space-y` to work
- Supabase clients: `server.ts` for Server Components, `client.ts` for Client Components, `admin.ts` for cron/privileged operations

## Maintenance Type Pattern
Each of the 6 types follows the same pattern:
1. **Page** at `src/app/[locale]/(app)/cars/[carId]/{type}/page.tsx` — client component with add/edit/delete
2. **DB table** with RLS policy joining through `cars.user_id`
3. **Translation keys** under its own namespace in messages JSON
4. **Status badge** on dashboard + car detail (except tires which use checkmark)
5. For date-based types: notification toggle fields (`notify_10_days`, `notify_5_days`, `notify_1_day`)

## Key Files
- `src/middleware.ts` — auth + i18n routing
- `src/lib/constants.ts` — maintenance type mappings
- `src/components/maintenance/record-list.tsx` — reusable list with edit/delete
- `src/components/dashboard/status-badge.tsx` — expiry status with success/warning/destructive/outline variants
- `supabase/migrations/001_initial_schema.sql` — full DB schema
- `src/types/database.ts` — Supabase types (must have Relationships arrays)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bozhidar-Vangelov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
