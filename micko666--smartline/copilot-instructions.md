## smartline

> Guidance for Claude Code sessions on this repository.

# CLAUDE.md

Guidance for Claude Code sessions on this repository.

## Commands

```bash
npm run dev          # Dev server at http://localhost:8080
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest (once)
npm run test:watch   # Vitest (watch)
```

Demo login: `demo@smartline.io` / `demo1234`

## Stack

**SmartLine** — restaurant operations platform.
- React 18 + TypeScript + Vite
- Zustand (store + localStorage persistence under key `smartline-v1`)
- Supabase (optional; toggled via `src/store/flags.ts` → `isSupabaseEnabled()`)
- Tailwind CSS + shadcn/ui, framer-motion, lucide-react
- Path alias: `@/` → `src/`

## Key Files

| Path | Purpose |
|------|---------|
| `src/domain/types.ts` | All TS interfaces — edit here first when adding fields |
| `src/domain/initialData.ts` | Default settings + seed data |
| `src/domain/orderMachine.ts` | Order status state machine |
| `src/store/index.ts` | Single source of truth; all business actions live here |
| `src/store/flags.ts` | `isSupabaseEnabled()` feature flag |
| `src/lib/supabase/mappers.ts` | DB row ↔ domain object mapping (snake_case ↔ camelCase) |
| `src/lib/supabase/queries/public.ts` | `fetchRestaurantByToken`, `submitBookingToSupabase` |
| `supabase/migrations/` | Sequential SQL migrations (apply via Supabase MCP) |
| `src/App.tsx` | All routes |
| `src/pages/customer/OrderPortal.tsx` | Public entry: mode selector → table picker / scheduling |
| `src/pages/customer/Menu.tsx` | Customer menu, cart, checkout (dine-in + takeaway + delivery) |
| `src/pages/customer/Receipt.tsx` | Post-order receipt page |
| `src/pages/admin/Orders.tsx` | Admin order management |
| `src/pages/admin/Settings.tsx` | Business settings |
| `src/components/layout/DashboardLayout.tsx` | Admin shell + notification badges |

## Routes

| Route | Surface | Auth |
|-------|---------|------|
| `/` | Login | Public |
| `/signup` | Signup | Public |
| `/order/:token` | OrderPortal (mode select → table/schedule) | Public |
| `/menu?t={id}&r={token}` | Dine-in customer menu | Public |
| `/menu?mode=takeaway&r={token}&date=…&time=…` | Takeaway menu | Public |
| `/menu?mode=delivery&r={token}&date=…&time=…&addr=…` | Delivery menu | Public |
| `/receipt/:id?r=…&mode=…&date=…&time=…` | Receipt | Public |
| `/book/:token` | Reservation booking | Public |
| `/dashboard` … `/settings` | Admin panel | AdminGuard |

## Customer Ordering Flows

**Dine-in**: `/order/:token` → pick table → `/menu?t={id}&r={token}` → cart → pay → receipt  
**Takeaway**: `/order/:token` → Takeaway → schedule (date + 15-min slot, min 30 min from now) → `/menu?mode=takeaway&date&time&r` → cart → pay → receipt  
**Delivery**: same as takeaway + mandatory address input → `/menu?mode=delivery&date&time&addr&r`

Receipt "Order More" reconstructs the full URL with all scheduling params — do not simplify it.

## Supabase RPCs

- `get_customer_menu(token)` — returns settings + menu items + tables in one call
- `atomic_checkout(...)` — serialised checkout; handles stock, reservations, order creation
- `submit_booking(...)` — creates `calendar_events` row
- All RPCs use `SECURITY DEFINER` + `SET search_path = public`
- Use `gen_random_uuid()` (pg_catalog), never `uuid_generate_v4()` (extensions schema — not on path)
- Non-UUID `table_id` strings (`takeaway`, `delivery`, `walk-in`) pass through a UUID regex guard

## Store Conventions

- No local state for business data — everything in `useStore`
- `useShallow` for all multi-key selectors
- `checkout()` is atomic — validates stock + reservations, deducts, creates Order + Receipt
- Table status only updated when `tableId` matches UUID regex (`/^[0-9a-f]{8}-…$/i`)
- `takeawayEnabled` / `deliveryEnabled` in `BusinessSettings` toggle channels

## Coding Conventions

- Keep explanations short unless teaching is requested
- Prefer targeted `Read` + `Edit` over full-file rewrites
- Run targeted tests before full suite: `npm test -- src/tests/store.test.ts`
- Migrations are numbered sequentially (`013_…`); apply via Supabase MCP `apply_migration`
- Domain types first → store actions → UI; never bypass the store from UI components
- shadcn/ui components in `src/components/ui/` — extend by composition, don't edit directly

## Current State (as of Apr 2026)

- Supabase backend live; local demo mode also works
- All three ordering channels functional end-to-end (dine-in, takeaway, delivery)
- Real-time sync via `useRealtimeCoordinator` in admin layout
- Calendar, Analytics, Stations, Ingredients, PrepTimes pages complete
- Orders page: today grid + carryover-active section (previous-day unfinished orders pinned at top) + historical accordion

## Remaining TODOs

1. ~~**Structured opening hours**~~ — ✅ Done. `businessHours: WorkingDay[]` in `BusinessSettings`; OrderPortal gates on `checkOpenStatus()`, slot picker filters to open windows. Settings page has full per-day editor.
2. **Delivery address map** — currently a plain text input; map integration planned.
3. ~~**Scheduled order visibility in admin**~~ — ✅ Done. `scheduledFor?: string` ("YYYY-MM-DD HH:MM") is a first-class field on `Order`; Menu.tsx sets it at checkout; Orders page shows a `CalendarClock` badge on takeaway/delivery cards.
4. **Auth hardening** — credentials stored in `smartline-accounts` localStorage array; ready to swap for real API calls in `login`/`signup` store actions.

---
> Source: [Micko666/SmartLine](https://github.com/Micko666/SmartLine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
