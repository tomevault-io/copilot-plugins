## investing-list

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev      # Start development server (http://localhost:3000)
pnpm build    # Production build
pnpm lint     # Run ESLint
```

Package manager: **pnpm**. No tests are configured.

## Git workflow

Always implement new features on a dedicated branch — never commit directly to `main`:

```bash
git checkout -b feature/<short-description>
```

Open a pull request to merge back into `main` when the feature is ready.

## Environment Setup

Copy `.env.local.example` to `.env.local` and fill in Supabase credentials:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
```

Without these vars, the app runs in local-only mode (no auth, data stored in localStorage).

## Architecture

**Next.js 16 App Router** app using React 19 and Tailwind CSS v4. No UI component library — all components are hand-written with inline Tailwind classes.

### Dual-storage pattern

Every context has two code paths:
- **Logged in** → reads/writes Supabase (cloud sync)
- **Logged out** → reads/writes `localStorage` via `src/lib/storage.ts`

This is the central architectural pattern. `isSupabaseConfigured` gates whether auth is even attempted.

### Context hierarchy (rendered in `AppShell`)

```
AuthProvider               ← Supabase auth state + recovery mode
  PortfolioListProvider    ← list of portfolios (CRUD)
    HoldingsProvider       ← all holdings aggregated for sidebar stats
      <page content>
        PortfolioProvider  ← per-portfolio holdings (CRUD), mounted per route
```

`HoldingsProvider` (`src/context/holdings-context.tsx`) loads a lightweight subset of all holdings to compute `PortfolioStats` shown in sidebar cards. `PortfolioProvider` (`src/context/portfolio-context.tsx`) loads full holding details for a single portfolio page. After mutations, pages call `reload()` from `HoldingsProvider` to refresh sidebar stats.

### Components

All components are in `src/components/` — hand-written with inline Tailwind, no UI library.

| Component | Purpose |
|---|---|
| `app-shell.tsx` | Root layout wrapper — mounts providers and renders `Header` + `PortfolioNav` around `{children}` |
| `header.tsx` | Top nav bar with app title, hamburger (mobile), and auth controls (Sign In / Sign Out) |
| `portfolio-nav.tsx` | Collapsible left sidebar with sortable portfolio list; contains the `NavPortfolioCard` sub-component (sidebar card with stats, gain/loss, core/satellite %, last-updated date) |
| `portfolio-card.tsx` | Larger card used on the home summary page; shows portfolio name, holdings count, total value, cost, and gain/loss |
| `portfolio-dialog.tsx` | Modal (`<dialog>`) for creating / editing / deleting a portfolio |
| `holding-dialog.tsx` | Modal (`<dialog>`) for creating / editing / deleting a holding; includes all fields: name, ticker, asset type, holding type, shares, average cost + currency, current price + currency, company ID, holding ID |
| `portfolio-summary.tsx` | Stat cards grid: Market Value, Total Cost, Gain/Loss, Return %; plus a core/satellite breakdown row |

Dialog pattern: both dialogs use the native `<dialog>` element (`showModal()` / `close()`), auto-focus the first input, close on Escape or backdrop click.

Responsive layout: mobile shows card-based views; desktop (`sm:` breakpoint) shows table layout for holdings. The sidebar (`PortfolioNav`) is hidden off-screen on mobile and slides in as a drawer overlay, always visible on `lg:`.

### Design tokens

Custom Tailwind color aliases used throughout (defined in the global CSS / Tailwind config):
- `text-gain` / `bg-gain` — green, positive returns
- `text-loss` / `bg-loss` — red, negative returns
- `text-foreground` / `bg-background` — theme-aware base colors

### Pages

| Route | File | Description |
|---|---|---|
| `/` | `src/app/page.tsx` | Aggregated summary across all portfolios using `PortfolioSummary` + `allHoldings` from `HoldingsProvider` |
| `/portfolio/[id]` | `src/app/portfolio/[id]/page.tsx` | Per-portfolio holdings view; desktop table + mobile card list; mounts `PortfolioProvider`; includes "Update NAV" button |

### API Routes

| Route | File | Description |
|---|---|---|
| `POST /api/fetch-nav` | `src/app/api/fetch-nav/route.ts` | Server-side proxy to Thailand SEC API; accepts `{ holdingId, navDate }`, retries up to 3 days back on 204, returns `{ lastVal, navDate }` or `{ lastVal: null, navDate: null }` |

### Data model (`src/types/portfolio.ts`)

- `Portfolio`: `{ id, name }`
- `Holding`: belongs to one portfolio; has `assetType` (stock/etf/mutual_fund/bond), `holdingType` (core/satellite), `shares`, `averageCost`/`currentPrice` each with a `Currency` (THB/USD), plus optional `ticker`, `companyId`, `holdingId`, `navDate` (last fetched NAV date string), and `updatedAt` (set by DB trigger)

### Currency handling

All display values are converted to THB using a **fixed rate** of 1 USD = 32 THB. The `toTHB(amount, currency)` helper in `src/lib/format.ts` handles this. Never compare costs across currencies without converting via `toTHB`.

### Supabase schema

SQL migrations are in `supabase/` (run manually via Supabase SQL Editor in order):
1. `001_create_holdings.sql` — holdings table with RLS
2. `002_add_portfolios.sql` — portfolios table; adds `portfolio_id` to holdings
3. `003_add_currency.sql` — currency columns
4. `004_add_holding_type.sql` — holding_type column
5. `005_add_updated_at_trigger.sql` — `updated_at` timestamp on holdings, auto-updated by trigger
6. `006_add_company_holding_ids.sql` — optional `company_id` and `holding_id` columns on holdings
7. `007_add_nav_date.sql` — optional `nav_date` TEXT column on holdings

DB column naming is snake_case; TypeScript field naming is camelCase. Context files contain mapper functions (`rowToHolding`, `rowToPortfolio`) for conversion.

Row Level Security is enabled — each user can only access their own rows via `auth.uid() = user_id`.

### Deployment

Deployed to **Netlify** via `@netlify/plugin-nextjs`. Config in `netlify.toml`.

---
> Source: [yutanui/investing-list](https://github.com/yutanui/investing-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
