## skywagev2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Skywage v2 — Airline salary calculator for Flydubai crew (CCM/SCCM). Calculates monthly salaries from roster uploads (CSV/Excel) or manual flight entry.

## Commands

```bash
npm run dev          # Start dev server (Turbopack)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint
```

No test runner is configured. Test files exist under `src/lib/__tests__/` and `src/lib/salary-calculator/__tests__/` for reference only.

## Tech Stack

- Next.js 16 (App Router) + React 19 + TypeScript 5
- Tailwind CSS 4 + Radix UI (shadcn/ui, new-york style) + lucide-react icons
- Supabase (PostgreSQL + Auth + SSR cookies via @supabase/ssr)
- Recharts for charts, XLSX + PapaParse for file parsing
- Deployed on Netlify

## Architecture

### Path Alias

`@/*` maps to `./src/*` (configured in tsconfig.json).

### Route Structure

- `src/app/(dashboard)/` — Protected routes (dashboard, friends, statistics, settings)
- `src/app/api/` — API route handlers (account, friends, debug)
- `src/app/auth/`, `src/app/login/`, `src/app/register/` — Public auth pages
- Auth enforced by `src/middleware.ts` (server-side) + `src/components/ProtectedRoute.tsx` (client-side)

### Provider Hierarchy (Dashboard Layout)

```
ProtectedRoute → AuthProvider → MobileNavigationProvider → ProfileProvider → FriendsProvider → Page
```

### Core Domains

| Domain           | Location                                 | Purpose                                   |
| ---------------- | ---------------------------------------- | ----------------------------------------- |
| Salary engine    | `src/lib/salary-calculator/` (38+ files) | Classification, calculation, parsing      |
| Database layer   | `src/lib/database/`                      | CRUD with audit trail                     |
| Position history | `src/lib/user-position-history.ts`       | Effective-dated position resolution       |
| Data hooks       | `src/hooks/`                             | Fetch flights, calculations, stats        |
| Contexts         | `src/contexts/`                          | Auth, Profile, Friends, MobileNav         |
| UI components    | `src/components/`                        | Dashboard, salary-calc, settings, friends |
| Types            | `src/types/salary-calculator.ts`         | Core domain types                         |

### Database Operation Pattern

Three wrappers in `src/lib/database/withDatabaseOperation.ts` provide consistent error handling:

- `withDatabaseOperation` — single item queries
- `withDatabaseArrayOperation` — array queries
- `withDatabaseVoidOperation` — mutations without return data

All database helpers in `src/lib/database/*.ts` use these wrappers.

### Effective-Dated Position History

`getUserPositionForMonth(userId, year, month)` in `src/lib/user-position-history.ts` is the **single source of truth** for position resolution. It queries `user_position_history` for the latest entry with effective date <= target month, falling back to `profiles.position`. All calculation workflows (upload, manual entry, recalculation, edits) must use this function — never read `profiles.position` directly for calculations.

### Month Indexing

- **Dashboard/UI**: 0-based (0 = January)
- **Database**: 1-based (1 = January)
- Conversion happens in hooks (e.g., `useFlightDuties`)

### Data Source Tracking

The `dataSource` field on flights tracks origin: `'csv' | 'manual' | 'edited' | 'cross_month_pairing'`. Audit snapshot fields (`position_used`, `hourly_rate_used`, `per_diem_rate_used`) on flights/calculations preserve historical accuracy.

### Salary Calculation Rules

Two rate eras determined by `getRatesForDate(year, month)`:

- **Legacy** (pre-July 2025): CCM basic 3,275 / SCCM basic 4,275
- **New** (July 2025+): CCM basic 3,405 / SCCM basic 4,446

Key rates: CCM 50 AED/hr, SCCM 62 AED/hr, Per diem 8.82 AED/hr of layover rest, ASBY/Recurrent = 4hr fixed duty. Deadhead (DHD) flights pay 50% of block time, not full duty hours.

Duty types: turnaround, layover, asby, sby, recurrent, rest, off, annual_leave, sick, business_promotion. Non-payable: off, rest, annual_leave, sby, sick.

Fixed allowances: Housing (4,000/5,000 AED CCM/SCCM) + Transport (1,000 AED).

### Supabase Schema (Key Tables)

- **flights** — Flight duties with dual old/new column schema for backward compatibility
- **layover_rest_periods** — Rest periods between layover flights with per diem
- **monthly_calculations** — Aggregated monthly salary breakdowns
- **user_position_history** — Effective-dated CCM/SCCM position records
- **flight_audit_trail** — Audit log capturing old/new data on mutations
- **profiles** — User profile with airline, position, nationality
- **friendships** — Friend requests and status

## Commit Convention

Use emoji-prefixed conventional commits:

```
✨ feat: description     # New feature
🐛 fix: description      # Bug fix
🎨 style: description    # UI/styling changes
🔨 refactor: description # Code refactoring
📝 docs: description     # Documentation
```

Use `git switch -c` for branches, not `git checkout`.

## Development Rules

- Prefer editing existing files over creating new ones
- Iterate on existing patterns before introducing new ones
- Avoid files over 200-300 lines — refactor when they grow beyond that
- Never mock data or add stubbing/fake data patterns
- Never overwrite `.env.local` without asking first
- Stay focused on the requested task — don't touch unrelated code
- Check for existing similar code before writing new code (avoid duplication)
- When fixing bugs, exhaust existing implementation options before introducing new patterns; if a new pattern is needed, remove the old one

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rprioli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
