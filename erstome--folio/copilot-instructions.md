## folio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Folio** is a personal finance tracker (Next.js 14 App Router + TypeScript) for investments, bank deposits, and retirement funds. All data is stored locally in a SQLite database via Prisma.

## Commands

```bash
# Install dependencies
npm install

# Initialize the database (creates prisma/dev.db)
npx prisma db push

# Development server (binds to 0.0.0.0 for LAN access)
npm run dev

# Production build
npm run build

# Lint
npm run lint

# Run tests (Vitest)
npm test

# Inspect database interactively
npx prisma studio

# After schema changes, regenerate Prisma client
npx prisma generate
```

Tests use Vitest (`npm test`); they live next to the code as `*.test.ts` files under `src/`.

## Architecture

### Data Layer

All data access is centralized in a single Next.js Server Actions file: **`src/app/actions.ts`**. This file handles every read and write operation — there are no separate API routes for data (the only route is the Google OAuth callback at `src/app/api/auth/google/callback/route.ts`).

- **`src/lib/db.ts`** — Prisma singleton (prevents multiple client instances in dev HMR).
- **`src/lib/server-services.ts`** — Isolates server-only Node.js imports (`yahoo-finance2`, `googleapis`, `fs`, `DB_PATH`) so they never leak into Client Components.
- **`src/lib/fx.ts`** — historical EUR/USD FX rates: daily ECB rates synced from the Frankfurter API into `HistoricalPrice` (symbol `EURUSD=X`, price = USD per 1 EUR), plus `convertCurrency`/`fxRateForDate` lookup helpers used by every currency-converting server action. Synced fire-and-forget on `/` and `/investments` page loads.

### Database Schema (`prisma/schema.prisma`)

Single `Asset` table with a `type` discriminator:
- `STOCK` / `ETF` / `CRYPTO` — market-priced assets with `Transaction` rows
- `DEPOSIT` — bank deposits; principal stored as `Transaction.price` (quantity = 1)
- `PENSION` — retirement funds; `manualPrice` field stores current unit price

`HistoricalPrice` stores end-of-month prices per symbol for the performance chart. `Setting` stores key-value app configuration (API keys, backup path, OAuth credentials) that can override environment variables.

### Quote Fetching (`getQuotes` in actions.ts)

Priority: **Twelvedata → FMP → Yahoo Finance → DB cache** (last known price).
An in-memory cache (5 min TTL) sits in front of all providers. ISINs are resolved to ticker symbols via Yahoo Finance search, with a 24h ISIN cache and a hardcoded `ISIN_OVERRIDES` map for known problematic ISINs (e.g. Xiaomi Frankfurt).

### Performance Calculation

Monthly performance uses the **Modified Dietz** method. `getAssetDetails` computes per-symbol performance; `getPortfolioPerformance` aggregates all assets into a global view. Historical prices are synced lazily in the background on each `/investments` page load (`syncHistoricalPrices`), throttled 2–5 s between symbols to avoid Yahoo Finance 429s.

### Pages & Routing

| Route | Component | Purpose |
|---|---|---|
| `/` | `src/app/page.tsx` | Global dashboard — net worth across all asset types |
| `/investments` | `src/app/investments/page.tsx` | Stock/ETF/Crypto portfolio |
| `/investments/[symbol]` | `src/app/investments/[symbol]/page.tsx` | Per-asset detail + monthly performance table |
| `/deposits` | `src/app/deposits/page.tsx` | Bank deposit tracker |
| `/pension` | `src/app/pension/page.tsx` | Retirement fund tracker |

Currency switching (`?currency=EUR` or `?currency=USD`) is a URL query param propagated between pages.

### Shared Types (`src/app/types.ts`)

Client-safe type definitions used by both Server Actions and Client Components: `TransactionData`, `DepositData`, `PensionData`, `Holding`.

### Statement Import Parsers (`src/lib/`)

Two broker import parsers, each with a matching pair of Server Actions in `actions.ts`:

- **`trade-republic-parser.ts`** — parses Trade Republic PDF bank statements. Uses `eval('require')('pdf-parse')` to bypass Next.js bundler restrictions on the pdf-parse module. Called by `parseTradeRepublicStatement` / `importTradeRepublicStatement`.
- **`xtb-parser.ts`** — parses XTB broker Excel (`.xlsx`) exports using the `xlsx` library. Reads the `Cash Operations` sheet (rows start at index 5). XTB tickers use exchange suffixes (e.g. `CDR.PT`) that are stripped to canonical symbols (`CDR`) via `XTB_EXCHANGE_SUFFIX_MAP` in `actions.ts`. Called by `parseXTBStatementAction` / `importXTBStatementAction`.

### Symbol Resolution

`getQuotes` in `actions.ts` normalises three symbol formats before fetching:
- **ISIN** (regex `/^[A-Z]{2}[A-Z0-9]{9}\d$/`) — resolved to ticker via Yahoo Finance search with 24h cache; hardcoded overrides in `ISIN_OVERRIDES` for known problem cases (e.g. Xiaomi Frankfurt `1810.F`).
- **XTB exchange-suffix** (regex `/^[A-Z0-9]+\.(US|PT|ES|IT|FR|BE)$/`) — normalised by `resolveXTBTicker` using `XTB_EXCHANGE_SUFFIX_MAP` / `XTB_TICKER_OVERRIDES`.
- **Plain ticker** — passed through unchanged.

### Portfolio State: Active vs. Closed

- `getPortfolio()` — returns current holdings (net quantity > 0).
- `getSoldPortfolio()` — returns fully-exited positions (net quantity = 0), shown in `ClosedPositionsTable` on the investments page.

### Client Components

All dialogs and interactive components in `src/components/` are `'use client'`. They call Server Actions directly (not via `fetch`). Notable ones:
- `DataManagementDialog` — local export/import, local path backup, and Google Drive backup/restore
- `ImportStatementDialog` / `ImportButton` — Trade Republic PDF statement import
- `ImportXTBDialog` / `ImportXTBButton` — XTB Excel statement import
- `HoldingsTable` — active holdings with inline asset name editing via `updateAssetName`
- `ClosedPositionsTable` — fully-exited positions (read-only)

### UI Stack

Tailwind CSS v4, Recharts for charts, Lucide React for icons. No component library — custom components in `src/components/ui/` (currently only `progress.tsx`).

### Environment Variables

| Variable | Source | Purpose |
|---|---|---|
| `DATABASE_URL` | `.env` | SQLite file path (e.g. `file:./dev.db`) |
| `GOOGLE_CLIENT_ID` | `.env` or DB Setting | Google OAuth for Drive backup |
| `GOOGLE_CLIENT_SECRET` | `.env` or DB Setting | Google OAuth for Drive backup |
| `NEXT_PUBLIC_BASE_URL` | `.env` or DB Setting | Callback URL base (e.g. `http://localhost:3000`) |
| `TWELVEDATA_API_KEY` | DB Setting only | Optional; preferred quote provider |
| `FMP_API_KEY` | DB Setting only | Optional; second-choice quote provider |

API keys for Twelvedata and FMP are stored in the `Setting` table via the Data Management dialog, not in `.env`.

---
> Source: [erstome/folio](https://github.com/erstome/folio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
