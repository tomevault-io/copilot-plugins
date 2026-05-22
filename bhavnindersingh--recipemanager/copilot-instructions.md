## recipemanager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Manager Conscious Cafe

A full-stack cafe management system with POS, KDS, stock management, and analytics.

**Stack:** React 18 (Netlify) + Supabase (PostgreSQL + Realtime + Storage)

---

## Development Commands

```bash
cd frontend
npm start        # dev server on http://localhost:3000
npm run build    # production build (output: frontend/build/)
npm test         # run tests (watch mode)
```

Deployment is via Netlify — push to `main` triggers a build automatically. Build base is `frontend/`, publish dir is `build/`, Node 20.

---

## Application Routes & Role Access

| Route | Component | Roles |
|-------|-----------|-------|
| `/pin-login` | PinLogin | public |
| `/dashboard` | Dashboard | admin, store_manager |
| `/pos` | POSPageNew | admin, cashier |
| `/kds` | KDSPage | admin, kitchen, cashier |
| `/bar-kds` | BarKDSPage | admin, bar, cashier |
| `/wood-fire-oven-kds` | WoodFireOvenKDSPage | admin, kitchen, cashier |
| `/server` | ServeOnlyPage | admin, server, cashier |
| `/table-self-orders` | TableSelfOrdersPage | admin, cashier |
| `/cloud-orders` | CloudOrdersPage | admin, cashier |
| `/manager` | RecipeManager | admin |
| `/create` | RecipeForm | admin |
| `/ingredients` | IngredientsManager | admin, store_manager |
| `/stock` | StockRegister | admin, store_manager |
| `/analytics` | Analytics | admin, cashier |
| `/data` | DataManager | admin, cashier |
| `/settings` | SettingsPage | admin |
| `/invoice/:orderId` | InvoicePage | public (customer WhatsApp link) |
| `/self-order` | SelfOrderPage | public (table QR scan) |

All heavy components are **lazy-loaded** via `React.lazy()`. CSS for POS, DataManager, RecipeManager is imported eagerly in `App.js` to prevent flash on first navigation.

`recipes` and `ingredients` are lazy-fetched in `App.js` — only loaded on first visit to `/manager`, `/create`, `/analytics`, or `/dashboard`. **POS never uses these** — it calls `getPOSRecipes()` which is a lightweight query without ingredient joins.

---

## Key Entry Points

| Feature | File |
|---------|------|
| App router + lazy loading | `frontend/src/App.js` |
| POS root | `frontend/src/components/POS/POSPageNew.js` |
| POS global state | `frontend/src/components/POS/context/POSContext.js` |
| Cart + billing UI | `frontend/src/components/POS/layout/RightCartPanel.js` |
| Table grid | `frontend/src/components/POS/views/TablesFullPageView.js` |
| Split bill | `frontend/src/components/POS/views/SplitBillView.js` |
| KDS | `frontend/src/components/KDS/KitchenDisplay.js` |
| Analytics (Sales Analysis) | `frontend/src/components/Analytics.js` |
| Order data page | `frontend/src/components/DataManager.js` |
| Order service | `frontend/src/services/orderService.js` |
| Table service | `frontend/src/services/tableService.js` |
| Payment service | `frontend/src/services/paymentService.js` |
| Recipe/ingredient CRUD | `frontend/src/services/supabaseService.js` |
| POS styles | `frontend/src/styles/POSNew.css` |

---

## Authentication

PIN-based — no Supabase Auth. Staff enter a 4-digit PIN which is verified server-side via the `verify_pin` RPC (bcrypt comparison in Postgres). The returned user profile is stored in `sessionStorage`.

- `authService.getCurrentUser()` — reads from sessionStorage
- `authService.hasRole('admin')` — checks current user's role
- `RoleBasedRoute` wraps all protected routes; unauthenticated → `/pin-login`
- After login, each role redirects to its default: `admin`→`/dashboard`, `cashier`→`/pos`, `kitchen`→`/kds`, `bar`→`/bar-kds`, `server`→`/server`, `store_manager`→`/dashboard`

---

## Dine-In Order State Machine

```
TABLE: available
  │ Staff taps table card in TablesFullPageView
  ▼
CONTEXT: selectedTable set, currentOrder fetched (or null if new)
  │ Staff adds items to cart → clicks "Send to Kitchen"
  ▼
TABLE: occupied  |  ORDER: status=pending, payment_status=unpaid
  │ KDS marks items preparing/ready/served  [auto-drives order.status]
  ▼
ORDER: status= cooking → ready → served
  │ Staff clicks "Generate Bill" in RightCartPanel
  │   → tableService.updateTableStatus(id, 'billed')
  │   → setSelectedTable({...table, status:'billed'})
  │   → print window opens
  ▼
TABLE: billed  |  ORDER: payment_status=unpaid
  [RightCartPanel shows: Pay Bill | Reprint | Reopen Table | Split Bill]

  ─── Reopen path ───────────────────────────────────────────────
  │ Staff clicks "Reopen Table"
  │   → tableService.updateTableStatus(id, 'occupied')
  │   → setSelectedTable({...table, status:'occupied'})
  └── Back to ORDER ACTIVE

  ─── Payment path ──────────────────────────────────────────────
  │ Staff clicks "Pay Bill" → selects method → "Process Payment"
  │   → paymentService.createPayment()  → DB: payment_status=paid
  │   → orderService.updateOrderStatus(id, 'completed')
  │   → orderService.getOrderById(id)   → setCurrentOrder(updated)
  ▼
ORDER: status=completed, payment_status=paid
TABLE: still billed  [Release Table button now visible]
  │ Staff clicks "Release Table"
  │   → tableService.clearTable(id)     → DB: table.status=available
  │   → resetState()                    → clears cart/table/order from context
  │   → setCurrentView('tables')
  ▼
TABLE: available  |  Context reset
```

---

## Status Enums (All Valid Values)

### `orders.status`
`pending` → `cooking` → `ready` → `served` → `completed`
Also: `cancelled`

> `pending`→`served` is driven by the KDS via `orderService.updateItemStatus()`.
> `completed` is set by the POS after payment via `orderService.updateOrderStatus(id, 'completed')`.

### `orders.payment_status`
`unpaid` → `partial` → `paid`

> Calculated by `paymentService.updateOrderPaymentStatus()`: sums all `payments` rows with `payment_status='completed'` and compares to `orders.total_amount`.

### `tables.status`
`available` | `occupied` | `billed` | `billing` (legacy) | `reserved`

> Only `available`, `occupied`, and `billed` are used by the current POS flow.

### `order_items.item_status`
`pending` → `preparing` → `ready` → `served`

> Driven entirely by the KDS.

### `payments.payment_status`
`pending` | `completed` | `failed` | `refunded`

> Always inserted as `completed` by the POS. `pending` is not used in current flow.

---

## POSContext State

| State | Type | Purpose |
|-------|------|---------|
| `cart` | `[{recipe, quantity, notes}]` | Items not yet sent to kitchen |
| `selectedTable` | `table \| null` | Currently active dine-in table |
| `currentOrder` | `order \| null` | Active order (with items + payment info) |
| `orderMode` | `DINE_IN\|TAKEAWAY\|SWIGGY\|ZOMATO` | Current order type |
| `currentView` | `'menu'\|'tables'\|'split-bill'` | Which view is rendered |

Key functions: `selectTable`, `resetState`, `switchTable`, `mergeOrders`, `showMessage`

---

## Service Layer

### `orderService.js`
- `createOrder(data)` — inserts order + items
- `updateOrderStatus(id, status)` — direct status update (no cascade)
- `updateItemStatus(itemId, status)` — updates item AND auto-promotes order status (KDS)
- `addItemsToOrder(orderId, items)` — append items to existing order
- `mergeOrders(sourceId, targetId)` — moves items, cancels source
- `getOrderById(id)` — full order with items + payments
- `getOrderStats({ dateFrom, dateTo })` — paginated order list for Analytics/DataManager
- `getTopItems(dateFrom, dateTo)` — order_items aggregated for menu performance (queries from `orders` parent, not `order_items` child — see PostgREST gotcha below)

### `tableService.js`
- `getTablesWithOrders()` — all tables joined with active orders (used by TablesFullPageView)
- `updateTableStatus(id, status)` — explicit status change
- `clearTable(id)` — sets status=`available` (table release)
- `subscribeToTables(callback)` — Supabase realtime channel

### `paymentService.js`
- `createPayment(data)` — inserts payment record, then calls `updateOrderPaymentStatus`
- `createSplitPayment(orderId, splits)` — inserts payment + split rows
- `updateOrderPaymentStatus(orderId)` — recalculates `paid_amount` + `payment_status` from all payments
- `getPaymentMethodBreakdown(dateFrom, dateTo)` — cash/card/UPI totals for Analytics

### `supabaseService.js`
- Recipe CRUD (`recipeService.*`), ingredient CRUD (`ingredientService.*`)
- `supabaseService.uploadImage(file)` — compresses via `imageUtils.js` then uploads to `recipe-images` bucket
- `get_all_recipes_sales()` RPC — aggregated sales counts per recipe

### `stockService.js`
- Stock transactions (purchase / adjustment / wastage / usage)
- `ingredient_stock` current levels; `stock_settings` reorder levels

### `authService.js`
- PIN verification via `verify_pin` RPC; sessionStorage read/write

### `whatsappService.js`
- Sends invoice links to customers via `POST /netlify/functions/send-whatsapp` → authkey.io API
- The Netlify function holds the API key — never in client code

---

## Realtime Channels

Three Supabase WebSocket channels are used. Always unsubscribe in `useEffect` cleanup:

```js
return () => orderService.unsubscribe(channel);
```

| Channel | Opened by | Watches |
|---------|-----------|---------|
| `tables-channel` | `tableService.subscribeToTables()` | `tables` — POS table grid |
| `cloud-orders-channel` | `orderService.subscribeToCloudCount()` in `App.js` | `orders` — sidebar delivery badge |
| KDS channel | `KitchenDisplay.js` | `order_items` + `orders` — kitchen screen |

`App.js` opens the cloud-orders channel once at root level and passes `selfOrderCount` down to POS so POS doesn't open a duplicate third channel.

---

## Database Key Tables & Triggers

**Core tables:** `orders`, `order_items`, `tables`, `payments`, `payment_splits`, `profiles`, `recipes`, `recipe_ingredients`, `ingredients`, `stock_transactions`, `ingredient_stock`, `stock_settings`

**Key auto-triggers:**
- `trigger_set_order_number` — generates `ORD-001`, `ORD-002`, …
- `trigger_update_stock_after_transaction` — keeps `ingredient_stock.current_quantity` in sync
- `trigger_auto_generate_recipe_sku` — assigns `FHB 001`-style SKU on recipe insert

**Helper RPCs:** `verify_pin`, `get_all_recipes_sales`, `get_recipe_sales_data`, `get_order_payment_summary`, `preview_next_sku`

---

## CSS Architecture

All POS styles live in `frontend/src/styles/POSNew.css`.
Global design tokens (colors, shadows, blur) are in `frontend/src/styles/shared.css` (`:root` block) — **always imported first** in `App.js`.
`POSNew.css` has its own `:root` block for POS-specific tokens (status colors, typography scale, border radii).

**Do not** add inline styles to POS components — use CSS classes defined in `POSNew.css`.

Each major non-POS component has its own CSS file (e.g. `Analytics.css`, `DataManager.css`). Use `an-` prefix for Analytics classes, `dm-` for DataManager, etc.

---

## Common Gotchas

1. **`orders.status` constraint** — valid values include `completed` (added in migration `20260320000000`). Do not use any other value.

2. **Walkout detection** — a walkout is `status === 'completed' && payment_status === 'unpaid' && amount_due >= 1`. Do NOT use `bill_generated_at` as a proxy — it is stamped on "Generate Bill" click for active orders and is not specific to walkouts.

3. **Supabase timestamps missing timezone** — Supabase `TIMESTAMP` columns return strings like `"2026-04-23T10:50:00"` with no `Z`. `new Date(naiveString)` parses as local time and breaks IST conversion. Always use `parseUTC(value)` from `frontend/src/utils/dateTime.js` which appends `Z` when no timezone marker is present.

4. **PostgREST embedded-filter direction** — when you need to filter by a parent table's column (e.g. `orders.created_at`), query from the **parent** (`orders`) and embed the children (`order_items`). Filtering by `order.created_at` when querying from `order_items` is silently ignored by PostgREST. See `getTopItems()` in `orderService.js` for the correct pattern.

5. **Scrollbar gutter** — `tables-grid-container` uses `overflow-y: scroll` + `scrollbar-gutter: stable` to prevent grid reflow on content change.

6. **Realtime channel cleanup** — always return the unsubscribe call from `useEffect` when using `tableService.subscribeToTables()`.

7. **`selectTable` is async** — it fetches the fresh order from DB. Don't assume `currentOrder` is set synchronously after calling it.

8. **IST everywhere** — all user-facing dates must use `formatISTDate` / `formatISTDateTime` / `formatISTTime` from `utils/dateTime.js` with `timeZone: 'Asia/Kolkata'`. Never display raw UTC or browser-local times.

---
> Source: [bhavnindersingh/RecipeManager](https://github.com/bhavnindersingh/RecipeManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
