## jollofexpress

> A production food ordering & delivery app for MyShawarma.express. Built with Next.js 15 (App Router), TypeScript, Supabase (PostgreSQL), and a Node.js thermal-printer sidecar. Customers order online, kitchen staff see orders on a Kanban display, orders are printed to a network thermal printer, and delivery is batched by time windows.

# JollofExpress / MyShawarma.express — Claude Code Guide

## Project Overview
A production food ordering & delivery app for MyShawarma.express. Built with Next.js 15 (App Router), TypeScript, Supabase (PostgreSQL), and a Node.js thermal-printer sidecar. Customers order online, kitchen staff see orders on a Kanban display, orders are printed to a network thermal printer, and delivery is batched by time windows.

## Tech Stack
- **Frontend/API:** Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS, Radix UI
- **Database:** PostgreSQL via Supabase (direct SQL + Supabase client)
- **State:** Zustand (cart), React Query (server state)
- **Payments:** Paystack
- **Printing:** ESC/POS over TCP to network thermal printer (port 9100, 80mm paper = 48 chars wide)
- **WhatsApp:** Baileys sidecar (`baileys-server.js`) on port 3001
- **Maps:** Mapbox GL + Leaflet + Turf.js

## Architecture

```
Customer Browser
  └─ Next.js App (port 3000)
       ├─ /app/api/           — API routes (orders, kitchen, print, admin, analytics)
       ├─ /app/kitchen/       — Kitchen Display System (KDS) — Kanban board
       ├─ /app/admin/         — Admin dashboard (analytics, menu management, settings)
       ├─ /app/checkout/      — Customer checkout flow
       └─ /app/menu/          — Customer-facing menu

Supabase (PostgreSQL)
  └─ Tables: orders, order_items, menu_items, item_variations, item_addons,
             delivery_batches, delivery_windows, print_queue, settings, ...

Print Worker (scripts/print-worker.js)
  └─ Polls print_queue → formats ESC/POS → sends over TCP to printer

WhatsApp Sidecar (baileys-server.js, port 3001)
  └─ WhatsApp notifications to customers
```

## Order Data Model (critical)

### `order_items` table
| Column | Type | Notes |
|---|---|---|
| `item_id` | UUID | Reference to menu_items |
| `item_name` | TEXT | **Snapshot** of item name at purchase time |
| `quantity` | INT | |
| `unit_price` | DECIMAL | Base price + variation + addons |
| `selected_variation` | JSONB | `{"name": "Protein", "option": "Hot Dog", "price_adjustment": 0}` |
| `selected_addons` | JSONB | `[{"name": "Extra Sauce", "price": 100, "quantity": 1}]` |
| `special_instructions` | TEXT | Per-item customer note |
| `subtotal` | DECIMAL | |

### Variation structure in cart vs DB
- **Cart (Zustand):** `selected_variation: { variation_name: "Protein", option: { name: "Hot Dog", price_adjustment: 0 }, quantity?: 1 }`
- **API request:** `{ name: "Protein", option: "Hot Dog", price_adjustment: 0, quantity?: 1 }`
- **DB (JSONB):** `{ "name": "Protein", "option": "Hot Dog", "price_adjustment": 0 }`

## Order Flow
1. Customer selects items + variations/addons → cart store (Zustand)
2. Checkout form → `POST /api/orders` → DB insert (orders + order_items)
3. Paystack payment initialized → customer redirected
4. Paystack callback → order confirmed → print job queued
5. Print worker polls `print_queue` → ESC/POS → TCP → thermal printer
6. Kitchen Kanban board polls for confirmed orders (real-time + polling)
7. Kitchen staff advance status: confirmed → out_for_delivery → completed

## Key Files

### Order Creation
- `app/api/orders/route.ts` — Main POST endpoint; validates, creates order + items, calls Paystack
- `components/checkout/checkout-form.tsx` — Builds the order payload from cart

### Kitchen Display
- `app/kitchen/page.tsx` — KDS page with Kanban board
- `components/kitchen/order-card.tsx` — Individual order card (summary + details modal)
- `app/api/kitchen/orders/route.ts` — Fetches confirmed/out_for_delivery orders

### Printing
- `lib/print/format-receipt.ts` — Converts `OrderWithItems` → `ReceiptData` → plain text (48 chars wide)
- `lib/print/escpos-generator.ts` — Generates raw ESC/POS commands
- `lib/print/print-processor.ts` — Fetches print_queue, sends to printer via TCP
- `scripts/print-worker.js` — Node.js daemon that drives print-processor

### Analytics
- `app/api/admin/analytics/top-items/route.ts` — Top items by quantity; groups by `order_items.item_name`
- `app/api/admin/analytics/overview/route.ts` — Revenue, order counts, AOV
- `app/api/admin/analytics/category-performance/route.ts` — Per-category stats

### Types & Data Access
- `types/database.ts` — All TypeScript interfaces (OrderWithItems, CartItem, etc.)
- `lib/supabase/service.ts` — Supabase service client (server-side, bypasses RLS)

## Deployment
See memory file for deploy procedure, PM2 setup, and secrets location.
- PM2 under `nodeapp` user: `sudo -u nodeapp pm2 restart all --update-env`
- Build: `npm run build` then restart PM2

## Coding Conventions
- API routes use Zod for request validation
- All DB writes go through Supabase service client (not public client)
- Prices are stored in **Naira** (not kobo) in the DB; Paystack converts × 100 internally
- `item_name` in `order_items` is a **snapshot** — decoupled from live menu changes
- ESC/POS receipt is 48 chars wide (80mm paper), uses ASCII approximations (NGN instead of ₦)

---
> Source: [resocorp/jollofexpress](https://github.com/resocorp/jollofexpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
