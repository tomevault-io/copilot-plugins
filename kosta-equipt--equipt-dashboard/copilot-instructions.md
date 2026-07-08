## equipt-dashboard

> The central daily analytics hub for Equipt. Kosta checks it every morning

# Equipt Dashboard — Project Brief for Claude Code

## What This Project Is

The central daily analytics hub for Equipt. Kosta checks it every morning
to see top-line social performance (Command Centre KPIs), trends over time,
and the content calendar for the week ahead.

All data flows live from the Equipt Social Media Tracker Google Sheet.
Google Sheets is the source of truth; the dashboard is a read-only view.

## Tech Stack

- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS with Equipt brand tokens (see `tailwind.config.ts`)
- Recharts for charts
- `googleapis` — server-side only, service-account auth
- Deployed on Vercel (Hobby / free tier)

## Data Flow

Google Sheet → `lib/sheets/client.ts` (service account, read-only) →
`lib/sheets/cache.ts` (5-min `unstable_cache` + `equipt-sheet` tag) →
`lib/sheets/adapters.ts` (typed parsers per tab) →
React Server Components (`app/page.tsx`, `app/planner/page.tsx`) →
Browser.

The **Refresh** button in the header calls the `refreshSheets` server
action in `app/actions.ts`, which calls `revalidateTag('equipt-sheet')`.
Next render bypasses the cache and pulls fresh from the Sheets API.

## Brand System

- **Midnight Navy** `#14304D` — primary
- **Amber Gold** `#D4A745` — accent only (10% use)
- **Refined Black** `#1C1C1E`
- **Warm Linen** `#F5F2ED`, **Bone** `#FAF7F2`, **Line** `#E6E1D8`
- Ratio: 60% neutrals / 30% navy-black / 10% gold
- Typography: Gotham (Arial fallback)
- Aesthetic: Apple-level minimalism, generous whitespace, editorial

Use the Tailwind tokens (`bg-bone`, `text-midnight`, `border-line`, etc.)
from `tailwind.config.ts`. Do not hand-code hex values in components.

## Sheet Tabs Consumed

| Tab name (env override) | Purpose |
|-------------------------|---------|
| `Dashboard` (`SHEET_TAB_COMMAND_CENTRE`) | KPI totals for the top row of the Analytics page |
| `Performance` (`SHEET_TAB_PERFORMANCE`) | Source for both the trend charts (Analytics) and the month-by-month History accordion (Planner). Every row = one published post with metrics. |
| `Content Calendar` (`SHEET_TAB_CONTENT_CALENDAR`) | Source for Week ahead on the Planner page. Only rows with a future-or-today date are shown. |

## Planner page data contract

- **Week ahead** = Content Calendar rows with `Date >= today`, sorted ascending.
- **History** = Performance tab rows, grouped by year-month (`dayLabel`, `monthKey`, `monthLabel` come from `lib/dates.ts`), sorted newest-first.
- No status column is required. A post is "upcoming" if its date is future; it becomes "history" once the Apps Script (`pullInstagramMetrics`) lands it in the Performance tab.
- Dates in the sheet are bare like `Apr 20` — year is inferred by `parseSheetDate`: assume current year, subtract 1 year if that would put the date in the future.

## Rules for Claude Code

- **Never** hardcode sample data as a long-term fix. The dashboard always reads from the sheet.
- **Never** commit `.env.local` or the service-account JSON.
- **Always** go through `cachedRange` in `lib/sheets/cache.ts` when reading a new range, so everything shares the 5-min TTL and the single revalidate tag.
- When adding a new widget:
  1. Add a parser in `lib/sheets/adapters.ts` that returns typed rows.
  2. Add a component under `components/analytics/` or a new folder.
  3. Keep it a Server Component unless it needs client interactivity.
  4. Use the Equipt tokens, not raw hexes.
- After every change, remind Kosta to run:
  `git add . && git commit -m "description" && git push`
- Read this file before starting any new task.

## Known Trade-offs

- Refresh cadence is 5 minutes. Click **Refresh** for an immediate re-pull.
- Free-tier Vercel (Hobby) is used — do not introduce paid dependencies.
- The adapter's KPI parser matches labels case-insensitively. If a label
  in the sheet changes, update `lib/sheets/adapters.ts`.

## Brand Context

- Company: Equipt (equipt.shop / equipt.qa)
- Tagline: Built to Perform
- Industry: Commercial gym equipment supply, installation & maintenance
- Location: Doha, Qatar + Dubai, UAE
- Managing Partner: Ivan Kruger (ivan@equipt.shop)
- Head of Sales & Ops: Yazan Hammad (yazan@equipt.shop)
- Marketing: Kosta Bouzopoulos (kosta@equipt.qa)

---
> Source: [kosta-equipt/equipt-dashboard](https://github.com/kosta-equipt/equipt-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
