## trading-journal

> A Next.js 16 trading journal application using Supabase for authentication and database. Built with TypeScript, React 19, and Tailwind CSS v4.

# AGENTS.md - Trade In Systems Development Guide

## Project Overview

A Next.js 16 trading journal application using Supabase for authentication and database. Built with TypeScript, React 19, and Tailwind CSS v4.

## Build, Lint, and Test Commands

```bash
# Development
npm run dev              # Start Next.js development server (localhost:3000)

# Production
npm run build            # Build for production
npm run start            # Start production server
npm run lint             # Run ESLint on the codebase

# No test framework is currently configured
```

## Code Style Guidelines

### TypeScript
- Strict mode is enabled in `tsconfig.json`
- Always use explicit types for function parameters and return types
- Use `import type` for type-only imports to improve performance

```typescript
// Good
import { supabase } from './supabase/client'
import type { TradeInsert, TradeUpdate } from './types'

// Bad
import { supabase, type TradeInsert } from './supabase/client'
```

### React/Next.js Conventions
- Use `'use client'` directive for client-side components
- Use functional components with TypeScript interfaces for props
- Name components using PascalCase
- Use file-based routing (App Router)
- Prefer server page wrappers for authenticated dashboards when initial data can be fetched on the server; pass initial props into a client component for interactive state

```typescript
interface TradeFormProps {
  trade?: Trade | null
  onClose: () => void
  onSuccess: () => void
  userId: string
}

export default function TradeForm({ trade, onClose, onSuccess, userId }: TradeFormProps) {
  // ...
}
```

### Naming Conventions
- Components: PascalCase (e.g., `TradeForm`, `TradeModal`)
- Functions/variables: camelCase (e.g., `getTrades`, `formData`)
- Types: PascalCase (e.g., `Trade`, `TradeInsert`, `System`, `SubSystem`)
- File names: kebab-case for non-components (e.g., `trade-form.tsx`, `trades.ts`)

### Imports Order
1. React/Next imports
2. External libraries (Supabase, etc.)
3. Internal imports (lib/, app/)
4. Type imports

```typescript
'use client'

import { useState, useEffect, useRef } from 'react'
import { createTrade, updateTrade } from '@/services/trade'
import { uploadScreenshot } from '@/services/upload'
import type { Trade, TradeInsert } from '@/services/trade'
```

### Path Aliases
Use `@/*` for absolute imports:
```typescript
import { getTrades } from '@/services/trade'
import { uploadScreenshot } from '@/services/upload'
import type { Trade } from '@/services/trade'
```

### Styling
- Use Tailwind CSS v4 utility classes
- Prefer semantic HTML elements
- Use `className` for Tailwind classes
- Ensure clickable buttons show pointer cursor (`cursor-pointer`) unless disabled

### Error Handling
- Throw Supabase errors directly after operations
- Catch errors in async handlers and set error state
- Display user-friendly error messages in UI

```typescript
// In data access functions
if (error) throw error

// In event handlers
try {
  await createTrade(formData)
  onSuccess()
} catch (err) {
  setError(err instanceof Error ? err.message : 'Failed to save trade')
}
```

### State Management
- Use `useState` for local component state
- Use Context API for global state (see `lib/AuthContext.tsx`)
- Prefer functional updates: `setFormData(prev => ({ ...prev, field: value }))`

### Database/Supabase
- Define types in respective service's `types.ts` (e.g., `services/trade/types.ts`)
- Use `TradeInsert` and `TradeUpdate` types for create/update operations
- `trades` supports both `system_id` and `sub_system_id` assignments
- Always filter queries by `user_id` for multi-user support

### ESLint Configuration
- Uses `eslint-config-next` with TypeScript support
- Run `npm run lint` before committing

### Formatting
- Use single quotes for strings
- Use 2-space indentation
- Add trailing commas in objects and arrays
- Use semicolons at the end of statements

### General Patterns
- Place related functions in dedicated module files (e.g., `services/trade/trades.ts`)
- Use section comments for grouping code (optional)
- Initialize optional fields with `null` rather than `undefined`
- Use `URL.revokeObjectURL()` to clean up object URLs in useEffect cleanup

## Project Structure

```
/app              # Next.js App Router pages
  /backtesting    # Backtesting sessions + theoretical trades page
    ImportBacktestingTradesForm.tsx # Backtesting sheet import modal with mapping preview
  /trades         # Trades page and components
    CloseTradeForm.tsx # Quick close-trade modal form (win/loss P&L input)
    ImportTradesForm.tsx # Sheet/CSV import modal with mapping preview
    /[tradeId]/page.tsx # Per-trade focus workspace (thinking discussion + chart snippet)
  /systems        # Systems + sub-systems management page
  /settings       # Account settings page (password update)
  /login          # Login page
  /signup         # Signup page
  layout.tsx      # Root layout
  page.tsx        # Home page
/lib              # Shared client-side code
  /supabase       # Supabase client setup
  AuthContext.tsx # Authentication context
/services         # Backend logic (Supabase operations)
  /backtesting    # Backtesting sessions + theoretical trades CRUD
    index.ts      # Re-exports
    backtesting.ts # Backtesting data operations
    types.ts      # Backtesting types
  /trade          # Trade CRUD operations
    index.ts      # Re-exports
    thinking-quotes.ts # Trade focus discussion CRUD helpers
    trades.ts     # Trade CRUD functions
    types.ts      # Trade + screenshot + thinking quote types
  /system         # System CRUD operations
    index.ts      # Re-exports
    systems.ts    # System + sub-system CRUD functions
    types.ts      # System + sub-system types
  /upload         # File upload operations
    index.ts      # Re-exports
/supabase
  /migrations     # SQL migrations (systems, sub-systems, backtesting, schema updates)
```

## Common Tasks

### Adding a new trade field
1. Add field to `Trade` type in `services/trade/types.ts`
2. Add field to `TradeInsert` and/or `TradeUpdate` if needed
3. Update `TradeForm.tsx` to include the new field

### Adding a new sub-system field
1. Add field to `SubSystem` type in `services/system/types.ts`
2. Add field to `SubSystemInsert` and/or `SubSystemUpdate` if needed
3. Update `/app/systems/page.tsx` sub-system modal/form

### Assigning trade to system and sub-system
1. Ensure `system_id` and `sub_system_id` exist in `services/trade/types.ts`
2. Update `/app/trades/TradeForm.tsx` selection logic
3. Keep sub-system options filtered by selected system

### Merging systems
1. Use merge flow on `/app/systems/page.tsx` to select source and target systems
2. Reassign all source system trades to target system before deletion
3. Delete source system only after successful trade reassignment

### Adding close-trade flow
1. Use `/app/trades/CloseTradeForm.tsx` for modal UI
2. Capture signed realized P&L (`+` win / `-` loss), then map to `realised_win` or `realised_loss` accordingly
3. Update `avg_exit` and `r_multiple` via `updateTrade` using the signed P&L to calculate R
4. Wire modal open/close state from `/app/trades/page.tsx`

### Mirroring live trades into backtesting sessions
1. Use `/app/trades/TradeForm.tsx` to allow optional backtesting session selection when creating or editing live trades
2. Always save the live trade first, then mirror to `backtesting_trades` through `services/backtesting/backtesting.ts`
3. Match an existing mirror by `asset` + `entry_price` + `stop_loss`; if multiple matches exist, update the most recent one
4. If no mirror exists and a session is selected, create one with `outcome_r` defaulting to `0` when `r_multiple` is missing
5. Keep close/edit flows in sync by updating mirrored `outcome_r` after live-trade updates

### Updating dashboard stats filters
1. Keep `/app/trades/page.tsx` filters driven by selected `system_id` / `sub_system_id`
2. If checkboxes are selected, calculate stats from selected rows only
3. If no checkboxes are selected, calculate stats from currently filtered rows
4. Keep period R card totals in sync for: today, this week, this month, last 90 days, this year
5. Keep best/worst performer cards (system, asset, weekday, hour bucket `HH:00`) calculated from the same filtered/selected rows
6. Show `Time #1` and `Time #2` in both best/worst performer cards
7. Keep `EV / Trade` displayed in R (not dollars)
8. Replace `Profit Factor` with `MEV` in stats cards
9. Compute EV explicitly as `EV = (Win Rate x Average Win) - (Loss Rate x Average Loss)` using R values (`r_multiple`)
10. Compute `MEV = EV x Number of Trades / Number of Months Traded` where months traded is elapsed whole months between first and last trade date (minimum `1`)
11. Support a trades date-range filter with presets (`Today`, `This Week`, `This Month`, `Last 30 Days`, `Last 90 Days`, `This Year`) plus custom `From` / `To`
12. Treat trades date-range presets and period R cards as UTC-based for consistent boundary behavior
13. Keep the live-trades date filter on its own dedicated row with a custom two-month calendar range picker, visible quick presets, and a status/actions block aligned with the rest of the filter bar
14. When a live-trades date range is active, mute the `Total R by Period` card with an overlay that explains the focused range and offers a one-click clear action

### Organizing live trades table
1. Keep ongoing trades (`avg_exit` is `null`) in a dedicated top table on `/app/trades/page.tsx`
2. Keep closed trades (`avg_exit` is not `null`) in a separate table below for history scanning
3. Include `Stop Loss` as a dedicated column in both ongoing and closed tables
4. Display trade date/time in `DD/MM/YY HH:mm` format in table rows
5. Support date sorting by clicking the `Date` header (cycle: default, ascending, descending)
6. Keep outcome filtering options (`All Trades`, `Won Trades`, `Lost Trades`) in the filters bar
7. Keep direction filtering options (`All Directions`, `Long Trades`, `Short Trades`) in the filters bar
8. Keep the date-range filter in the filters bar and apply it before table sorting
9. Add a `No System` option in the system filter to show trades with no assigned `system_id`
10. Preserve per-row actions (`Close`, `Decisions`, `Chart`, `Edit`, `Delete`) across both tables
11. Keep row selection for ongoing and closed trades compatible with stats cards and filtered rows
12. Show a bulk-actions toolbar for selected ongoing and closed trades with `Edit Selected`, `Delete Selected`, plus `Select all` / `Unselect all` for the visible filtered rows
13. Bulk edit should support updating `system_id` (including `No System`) and `risk`; when risk changes, recalculate `r_multiple` from each trade's existing realized outcome

### Importing trades from sheets/files
1. Use `/app/trades/ImportTradesForm.tsx` for CSV/TSV/XLS/XLSX upload + mapping UI
2. Let users configure `rowsToSkip` and per-field column mappings with sample preview
3. Accept optional `trade_time` and `system_name` mappings during import
4. If `system_name` is mapped and system doesn't exist, create it with empty rules and assign `system_id`
5. Ignore non-data rows and log invalid rows in console with reason
6. If an imported trade already exists, update the existing record instead of creating a duplicate (supports re-import corrections like date fixes)
7. Skip file-internal duplicates while importing
8. Support both separate date/time columns and combined date-time in one column (users can map both fields to same source column)
9. Keep `trade_date` mapping optional during import (default missing dates to today)
10. Support outcome import modes for live trades: `Auto-detect`, `Signed P&L column`, and `Separate Win / Loss columns`
11. If `realised_pnl` is mapped or win/loss both point to the same source column, treat it as signed P&L and split positive values into `realised_win` and negative values into `realised_loss`
12. Parse signed P&L strings with common currency/unit suffixes (for example `-0.70 USDT`) during live trade import
13. If `avg_exit` is missing during live trade import, use `avg_entry` as a placeholder close price and append note marker `[Imported] Exit missing; placeholder avg_exit=avg_entry`

### Backtesting sessions and theoretical trades
1. Use `/app/backtesting/page.tsx` for session management and theoretical trade journaling
2. Backtesting sessions can link to existing `system_id` or create a new system during session creation
3. Backtesting trades store theoretical results in `outcome_r` (Profit in R), not realized dollar PnL
4. Use `/app/backtesting/ImportBacktestingTradesForm.tsx` for bulk importing theoretical trades with mapping and preview
5. Keep `trade_date` mapping optional during backtesting import (default missing dates to today)
6. Backtesting trade form should prefill `asset` from localStorage key `trade_form_last_asset` and persist the latest saved asset for faster repeated journaling
7. In backtesting trade modal, the `Loss` helper can copy `stop_loss` into `target_price` for quick failed-setup journaling
8. Backtesting trades list should be ordered by `created_at` descending (latest added first)
9. Backtesting table supports row selection; when rows are selected, session/performance stats should calculate from selected rows only (otherwise all rows)
10. Session stat cards include `Trades / Week`, calculated as `totalTrades / distinctMonthWeekBuckets` where each bucket is week-in-month within the same `YYYY-MM`
11. Session stat cards also include `Avg Win (R)` and `Avg Loss (R)`
12. Replace `Profit Factor` with `MEV` in session stat cards
13. Compute EV explicitly as `EV = (Win Rate x Average Win) - (Loss Rate x Average Loss)` using `outcome_r`
14. Compute `MEV = EV x Number of Trades / Number of Months Traded` where months traded is elapsed whole months between first and last trade date (minimum `1`)
15. Backtesting performance cards include best/worst `Asset`, plus day/time rankings with #1 and #2 entries (day names and `HH:00` time buckets)
16. Add-trade modal supports a `Keep modal open after adding` checkbox; on add success it keeps modal open and clears only entry/SL/TP (and recalculated outcome field)
17. Disable add/update submit while AI screenshot extraction is running
18. Show a subtle session status line for `Time spent backtesting` based on elapsed time between first and last trade `created_at`
19. Support date sorting in the backtesting trades table by clicking the `Date` header (default, ascending, descending)
20. Support direction filtering in backtesting sessions (`All Directions`, `Long Trades`, `Short Trades`) and keep stats aligned with filtered/selected rows

### Exporting backtesting sessions
1. Export is available from `/app/backtesting/page.tsx` as `Export CSV`
2. Current export file contains trade records only (no summary stats block)
3. Use human-friendly trade headers: `Trade Date`, `Trade Time`, `Asset`, `Direction`, `Entry Price`, `Stop Loss`, `Target Price`, `Outcome (R)`, `Notes`
4. Exclude technical IDs/timestamps from exported rows (`trade_id`, `created_at`)
5. Keep CSV spreadsheet-friendly for locale decimals by using `sep=;` and semicolon-delimited columns

### Viewing trade charts
1. Live trades use `/app/trades/TradeChartView.tsx`; backtesting trades use `/app/backtesting/BacktestingTradeChartView.tsx`
2. Fetch candles from Binance REST klines API with symbols normalized to `BINANCE:<BASE>USDT`
3. Treat `trade_date` and `trade_time` as UTC when anchoring chart context and entry markers
4. Persist last selected chart timeframe in localStorage key `trade_chart_timeframe` and fallback to `1h`
5. Support loading more context candles and keep entry-time vertical marker + entry/stop/exit lines visible
6. Display a volume histogram beneath candles in live, backtesting, and trade-focus chart views
7. On `/app/trades/page.tsx`, open chart views as multiple floating widgets (draggable, resizable, independently closable)

### Trade focus thinking workflow
1. Open per-trade focus from `/app/trades/page.tsx` row action `Decisions` (route `/app/trades/[tradeId]/page.tsx`)
2. Keep the discussion timeline sorted by `created_at` ascending (oldest to newest)
3. Store discussion entries in `trade_thinking_quotes` (migration `supabase/migrations/005_create_trade_thinking_quotes.sql`)
4. A discussion entry supports text and/or image (`quote_text`, `image_storage_path`, `image_filename`)
5. Allow posting only while the trade is ongoing (`avg_exit` is `null`); closed trades are read-only
6. Support image attachments via file picker and clipboard paste (`Cmd+V` / `Ctrl+V`), with premium gating for image upload
7. Keep discussion sidebar sticky on desktop and place composer at the bottom of the sidebar for continuous journaling

### Premium billing and feature gating
1. Subscription state is stored in `user_subscriptions` (see `supabase/migrations/004_create_user_subscriptions.sql`)
2. Stripe checkout is handled by `POST /api/stripe/checkout`; Stripe webhooks are handled by `POST /api/stripe/webhook`
3. Crypto checkout is handled by `POST /api/crypto/checkout`; NOWPayments IPN is handled by `POST /api/crypto/webhook`
4. Crypto payments currently use USDT on Polygon (`pay_currency=usdtmatic`) with manual renewal (no auto-recurring wallet charges)
5. Stripe checkout and customer portal are currently disabled for crypto-only billing mode; Stripe webhook handling remains in place for legacy events
6. Home-page pricing and signup plan selection use `PREMIUM_THREE_MONTH_PRICE_USD` and `PREMIUM_ANNUAL_PRICE_USD`; crypto checkout can override with `NOWPAYMENTS_PREMIUM_THREE_MONTH_PRICE_USD` and `NOWPAYMENTS_PREMIUM_ANNUAL_PRICE_USD` (legacy `*_MONTHLY_*` / `*_TWO_MONTH_*` names remain supported for compatibility)
7. The `monthly` crypto plan grants 3 months of premium access per successful payment; `annual` grants 12 months
8. Customer self-service billing is handled by `POST /api/stripe/portal`
9. Checkout returns to the home-page pricing flow with `/?intent=premium&checkout=success#pricing` on success and `/?intent=premium&checkout=cancelled#pricing` when canceled (`/premium/*` routes only redirect for compatibility)
10. Crypto webhook signature uses `NOWPAYMENTS_IPN_SECRET`; API requests use `NOWPAYMENTS_API_KEY`
11. Checkout quotes use the configured plan price directly (no additional quote buffer)
12. Partial payment handling uses tolerance bands: auto-accept (`NOWPAYMENTS_AUTO_TOLERANCE_FLAT`, `NOWPAYMENTS_AUTO_TOLERANCE_PERCENT`) and manual-review zone (`NOWPAYMENTS_REVIEW_TOLERANCE_FLAT`, `NOWPAYMENTS_REVIEW_TOLERANCE_PERCENT`)
13. For stablecoin checkout, avoid cross-currency conversion rails (e.g. `price_currency=usd` + `pay_currency=usdttrc20`) unless explicitly intended; prefer same-currency rails to reduce hidden conversion fees
14. Before shipping payment config changes, create a fresh test invoice and verify expected amount/fee sanity against target price to avoid expensive misconfiguration
15. Premium status for UI gating is fetched via `GET /api/subscription/status` and consumed through global provider `lib/PremiumContext.tsx` as a single shared fetch per authenticated session via `lib/usePremiumAccess.ts`
16. Premium-only features: screenshot upload, importing live trades, mirroring live trades to backtesting, one-click/multi-widget trade chart view, and creating more than 2 systems
17. Locked features remain visible; non-premium users are redirected to home-page pricing with `/?intent=premium&feature=<feature>#pricing`
18. New visitors entering the premium funnel sign up first, then land on `/signup?step=plan` (or `/signup?intent=premium&step=plan`) to choose `Free`, `3-Month`, or `Annual` before app entry or checkout
19. Keep `/api/subscription/status` as a fast DB-backed read; entitlements are reconciled by webhook flows instead of per-request provider checks
20. Admin role is stored on `user_subscriptions.app_role` (`user` or `admin`)
21. Admins can generate single-use premium invite links from Settings (`/api/admin/invites`)
22. Invite signup stores the token in auth metadata; trial is granted on first login and backdated to account creation time using `INVITE_PREMIUM_DAYS` (default `15`)

### AI screenshot trade prefill
1. Use `POST /api/ai/extract-trade-from-image` to parse a TradingView screenshot into trade field suggestions
2. Keep extraction as prefill-only assistance; user must review/edit and explicitly save the trade
3. Return `null` for missing/uncertain fields and surface warnings instead of guessing values
4. Gate this feature for premium users and redirect free users with `feature=ai-screenshot-import`
5. For chart screenshots, prioritize extracting `entry` and `stop_loss` from the TradingView position tool boundaries (split line + red-zone outer edge) and ignore current-price labels/dotted lines
6. Backtesting AI prefill uses `extraction_context=backtesting` so extraction can also include `target_price` from the green-zone outer edge of the same active position tool
7. Live AI prefill should skip `target_price` extraction (`extraction_context=live`) and rely on normal live-trade calculations for outcome fields
8. Backtesting trade modal supports both image upload and clipboard paste flows for AI prefill
9. Direction and `r_multiple` should be concluded/calculated by app logic from extracted prices, not trusted from model output
10. Configure OpenRouter with `OPENROUTER_API_KEY`
11. Configure the AI screenshot extraction model with `OPENROUTER_MODEL`
12. Default fallback model is `qwen/qwen3-vl-8b-instruct` when no model env vars are set

### Creating a new page
1. Create directory in `/app` (e.g., `/app/analytics`)
2. Create `page.tsx` in the new directory
3. Use appropriate layout (or create custom)

### Running the app locally
```bash
npm run dev
```

Then visit http://localhost:3000

## SSR Notes

- `/app/trades/page.tsx` is a server wrapper that preloads user, trades, systems, and sub-systems, then renders `/app/trades/TradesClient.tsx`
- `/app/backtesting/page.tsx` is a server wrapper that preloads user, systems, sessions, and initial session trades, then renders `/app/backtesting/BacktestingClient.tsx`
- Keep modal state, sorting, filters, AI actions, and other highly interactive behavior in the client shells; keep initial auth/data fetches in the server wrappers when possible

## SEO Notes

- Global SEO defaults live in `/app/layout.tsx` and use canonical domain `https://tradeinsystems.com`
- Keep `/app/robots.ts` and `/app/sitemap.ts` updated whenever public marketing routes are added
- Private app areas (`/trades`, `/backtesting`, `/systems`, `/settings`) should remain `noindex`
- Auth routes (`/login`, `/signup`) should remain `noindex,follow`
- Homepage structured data lives in `/app/page.tsx` and currently includes `Organization`, `SoftwareApplication`, and `FAQPage`

---
> Source: [HatimLagzar/trading-journal](https://github.com/HatimLagzar/trading-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
