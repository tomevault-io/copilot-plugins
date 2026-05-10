## zmart-deli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Zmart Deli Order Management System** — A web app for a convenience store deli that digitally replaces the manual/paper process of communicating orders from cashiers to deli/pizza cooks.

### The Problem It Solves
Cashiers currently hand-write or verbally relay orders to the deli cook. This app replaces that with a real-time digital order queue.

---

## User Roles & Devices

| Role | Device | Notes |
|------|--------|-------|
| Cashier (x2) | Personal phone (iOS/Android) | Register 1 or Register 2 |
| Deli/Pizza Cook | Fixed iPad | Shared station device; helpers may take over |

---

## Access Control
- No formal login. On first open, user enters: **name**, **station** (Register 1, Register 2, or Deli Cook), and a **global PIN** shared across all staff.
- This is saved to `localStorage` — returning users skip the gate.
- Language preference is also saved to `localStorage`.

---

## Menu

### Deli
- **Items**: Chicken Wings (Regular or Cajun), Chicken Tenders (Regular or Cajun), Chicken Legs (Regular or Cajun), Fish Sandwich
- **Quantity**: Any whole number (1, 2, 3...)
- **Combo option**: Any item can be ordered as a combo (= 3x that item + fries + can of drink). Multiple combos allowed (e.g. 2x Cajun Wing Combo).

### Pizza
- **Crust**: Regular or Thin
- **Size**: One size only
- **Toppings**: Pepperoni, Beef, Bacon, Bell Peppers, Jalapeño, Onions, Black Olives, Banana Peppers, Mushroom, Sausage
- **Half-and-half**: Toppings can be assigned to Left half, Right half, or Whole pizza
- **Extra**: Any topping can be marked as "extra"

---

## Order Lifecycle

1. Cashier builds order (customer name + items) and submits
2. Order appears instantly on the cook's screen
3. Cook marks order as **Done** when complete
4. **Edit**: Long-press on an order (rare use case) — available to cashier or cook
5. **Cancel**: Available to cashier or cook — very rare

Each order displays: customer name, items, cashier name + register, timestamp.

---

## UI/UX Requirements

- **Mobile-first, large UI**: Big tap targets — users have dirty/wet hands
- **Minimal scrolling**: Cook's screen must show active orders without scrolling; cashier order entry must be completable in as few taps as possible
- **RTL layout** required when Arabic is selected

---

## Internationalization

- **Languages**: English, Arabic, Spanish
- Switchable per user at any time; saved to `localStorage`
- Full UI translation required in all three languages (including order item names)

---

## Notifications

- Push notifications required — must wake up the device even when the app is backgrounded or the screen is off
- Cook is the primary notification target (new order arrived)
- Cashier may also need notification when an order is marked done

---

## Stack

### Frontend
- **React + Vite** — component-based UI, fast dev/build
- **Tailwind CSS** — utility-first styling, easy large-touch-target design
- **i18next** — i18n with RTL support for Arabic
- **PWA** — optional; prompt to install but not required. Enables home screen install and push notifications on iOS 16.4+

### Backend (self-hosted VPS)
- **Node.js + Fastify** — REST API
- **PostgreSQL** — order persistence
- **Server-Sent Events (SSE)** — real-time push from server to clients (cook's screen). Cashier order submission is a plain HTTP POST.

### Push Notifications
- **Web Push API with VAPID keys** via `web-push` npm library — fully self-hosted. Your server sends pushes; browsers route through their own infrastructure (no Firebase account needed).
- **Fallback**: loud in-app sound + vibration when app is open/foregrounded (covers iOS Safari non-PWA users)

### Hosting
- Self-hosted on personal VPS
- Frontend: served as static build (Nginx or Caddy)
- Backend: Node.js process (PM2 for process management)

---

## Commands

```bash
# Development (run both in separate terminals)
npm run dev:client        # Vite dev server at http://localhost:5173
npm run dev:server        # Fastify server at http://localhost:3000 (node --watch)

# Production build
npm run build             # Outputs to client/dist/

# Database setup (run once)
psql $DATABASE_URL -f server/src/db/schema.sql

# Generate VAPID keys (run once, paste into server/.env)
node -e "import('web-push').then(wp => console.log(wp.default.generateVAPIDKeys()))"
```

Copy `server/.env.example` → `server/.env` and fill in values before starting the server.

---

## Architecture

```
client/src/
  i18n/          # en/ar/es JSON translations + i18next setup (RTL dir toggled here)
  hooks/
    useIdentity  # localStorage gate — name, station, PIN result
    useOrders    # SSE listener; reconnects on drop; triggers in-app sound+vibrate
  views/
    GateView     # First-open PIN + name + station selector
    CashierView  # Order builder (DeliPanel + PizzaPanel sub-panels)
    CookView     # Live order queue; long-press to cancel
  components/
    LanguageSwitcher  # EN / AR / ES toggle, updates html[dir]
  App.jsx        # Routes: no identity → Gate, cook → CookView, cashier → CashierView

server/src/
  db/
    pool.js      # pg Pool from DATABASE_URL
    schema.sql   # orders + push_subscriptions tables
  sse/
    broker.js    # In-memory Set of SSE response objects; broadcast(event, data)
  routes/
    auth.js      # POST /api/auth/verify — checks APP_PIN env var
    orders.js    # CRUD for orders + web-push on new order + push subscription endpoint
    sse.js       # GET /sse/orders — establishes SSE stream
  index.js       # Fastify bootstrap, CORS, VAPID config
```

### Key data flow
1. Cashier POSTs to `/api/orders`
2. Server inserts to Postgres, calls `broadcast('order:new', order)` (SSE) and `webpush` to all cook subscriptions
3. Cook's `useOrders` hook receives the SSE event → React state update → card appears instantly
4. Cook taps Done → `PATCH /api/orders/:id/done` → server broadcasts `order:deleted` → card removed

### Key Constraints
- iOS push notifications only work if PWA is added to home screen (iOS 16.4+) — acceptable since cook uses a fixed iPad that can be set up once
- No third-party services (no Firebase, no external notification platforms)

---
> Source: [RyanX5/zmart-deli](https://github.com/RyanX5/zmart-deli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
