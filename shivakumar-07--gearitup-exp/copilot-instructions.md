## gearitup-exp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AutoSpace** is a B2B2C SaaS platform for India's auto parts retail market. It serves two personas simultaneously:
- **Shop Owners**: Use the integrated ERP/POS to manage inventory, billing, credit (udhaar), and workshop jobs
- **Vehicle Owners / Mechanics**: Browse parts with fitment guarantees, compare prices across local shops, buy with hyperlocal delivery

## Commands

```bash
npm run dev       # Start dev server (Vite + HMR)
npm run build     # Production build → dist/
npm run lint      # ESLint
npm run preview   # Preview production build locally
```

No test framework is configured.

## Architecture

### Dual-Mode Application

`App.jsx` renders either the **Marketplace** (B2C) or **Shop ERP** (B2B) based on a `mode` state flag. Both modes share the same global React Context store.

**Mode switching** is done in `App.jsx`. The marketplace is accessible to customers; the ERP is for shop owners.

### State Management

All state lives in a single React Context (`src/store.js`). Key entities:

- `products` — master product catalog
- `movements` — **immutable ledger** of all stock transactions (PURCHASE, SALE, ESTIMATE, RETURN_IN, RETURN_OUT, DAMAGE, THEFT, TRANSFER, RECEIPT, PAYMENT, ADJUST, etc.)
- `parties` — customers/suppliers with credit ledger (udhaar)
- `shops` — multi-shop support (3 demo shops)
- `orders` — marketplace B2C orders
- `jobCards` — workshop management
- `cart`, `selectedVehicle`, `auditLog`, `receipts`

All state is persisted to `localStorage` with a `vl_` prefix.

### Immutable Ledger Pattern

Stock is **never edited directly**. Every stock change is a transaction appended to `movements`. Current stock is always computed:

```
stock = opening + purchases - sales - adjustments - damage - theft
```

This is the core architectural invariant — do not break it by mutating stock directly.

### Routing

There is no React Router. Navigation is state-based: a `currentPage` string in `App.jsx` controls which page component renders. Shop ERP keyboard shortcuts: `D`=Dashboard, `I`=Inventory, `P`=POS, `A`=Parties, `W`=Workshop, `H`=History, `R`=Reports, `O`=Orders.

### Key Source Files

| File | Purpose |
|------|---------|
| `src/App.jsx` | Main app, routing, mode switching (34KB) |
| `src/store.js` | Global Context + all state reducers |
| `src/theme.js` | Design system: colors, typography, animations |
| `src/utils.js` | Seed data + shared utilities (39KB) |
| `src/vehicleData.js` | Vehicle compatibility database (26KB) |
| `src/barcode.js` | Barcode scanner integration |
| `src/marketplace/api/engine.js` | Search & fitment matching logic |
| `src/marketplace/api/mockDatabase.js` | Mock backend data |

### Design System (`src/theme.js`)

Dark-mode only. Color palette:
- Primary: Amber `#F59E0B`
- Success: Emerald `#10B981`
- Danger: Crimson `#EF4444`
- Info: Sky `#38BDF8`
- Background: Deep Navy `#0A0F1D`

Styling is done with **inline styles** referencing `theme.js` constants. There is no CSS framework (no Tailwind, no MUI). Reusable UI primitives live in `src/components/ui/` (Badge, Btn, Modal, Input, Select, StatCard, Toast, etc.).

### Fitment System

Vehicle compatibility is the core UX differentiator. `src/vehicleData.js` stores compatibility data by make/model/year/fuel/variant. The marketplace forces vehicle selection before browsing so all products shown are guaranteed to fit. `src/marketplace/api/engine.js` handles the matching logic.

### Indian Retail Specifics

- **Udhaar (credit)**: Built into `PartiesPage` — track receivables/payables, send WhatsApp reminders
- **Multi-tender billing**: POS supports splitting payment across cash + UPI + credit in one bill
- **GST compliance**: Every transaction records GST; exportable for GSTR-3B/GSTR-1
- **Barcode scanning**: Already implemented in `src/barcode.js`, used in POS and Inventory

## Spec Documents

For deep product/architecture context, consult:
- `DEEP_RESEARCH.md` — 13-section technical guide on e-commerce logic, ERP requirements, UI/UX design system
- `PROMPT_SPEC.md` — Executive product spec: value props, use cases, edge cases, architecture requirements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shivakumar-07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
