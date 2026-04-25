## jaliscotiendamexicana

> Point of Sale system for **Jalisco Tienda Mexicana**, a Mexican grocery store and restaurant. The app runs as both an Electron desktop app (production) and a browser preview (GitHub Pages demo). It handles grocery/restaurant transactions with barcode scanning, item grid selection, customer lookup, payment processing, held transactions, and receipt printing.

# Jalisco Tienda Mexicana POS

## Project Overview

Point of Sale system for **Jalisco Tienda Mexicana**, a Mexican grocery store and restaurant. The app runs as both an Electron desktop app (production) and a browser preview (GitHub Pages demo). It handles grocery/restaurant transactions with barcode scanning, item grid selection, customer lookup, payment processing, held transactions, and receipt printing.

**Live preview:** https://graphicaljerry.github.io/Jaliscotiendamexicana/
**Figma designs:** https://www.figma.com/design/rRcPCDbbV4Vv37GRzVdUu8/Jalisco-Tienda-Mexicana

### Figma Key Frames
- **Main POS screen:** node-id=6637-3814
- **Sidebar open:** node-id=6644-4602
- **Sidebar panel detail:** node-id=6644-4954
- **Payment modal:** built via use_figma on POS System page

## Tech Stack

- **Framework:** React 18 + Vite 6
- **Desktop:** Electron 33 (with electron-builder for packaging)
- **State:** Zustand 4 (single store: `transactionStore.js`)
- **Routing:** React Router v6 (BrowserRouter with `v7_startTransition` and `v7_relativeSplatPath` future flags — previously HashRouter, migrated to BrowserRouter)
- **Styling:** Plain CSS with CSS variables (NO Tailwind, NO CSS modules)
- **Database:** better-sqlite3 (Electron only; browser uses mock API)
- **Printer:** node-thermal-printer (Electron only)
- **Inventory:** 17,698 items loaded from `src/data/inventory.json` (TGS export, ~1.5MB)

## Architecture

### File Structure
```
src/
  App.jsx                    # BrowserRouter with /, /admin, /payment-preview routes
  main.jsx                   # Entry point, installs mock API for browser
  api-mock.js                # Browser mock: categories, items, customers, barcode lookup
  data/inventory.json        # 17,698 store items (barcode, name, price, tax, ebt)
  stores/transactionStore.js # Zustand store — ALL transaction state and computed values
  hooks/
    useBarcodeScanner.js     # USB barcode scanner detection (rapid keystroke pattern)
    useKeyboardShortcuts.js  # F1-F12 + Escape key mappings
  components/
    layout/
      TopBar.jsx/css         # Dark #121212 header bar (logo, txn#, Item Grid, Admin)
      TotalsBar.jsx/css      # Bottom totals (Paid, Change, Items, Sub Total, Tax, EBT, Grand Total)
      Logo.jsx               # Color SVG logo (jalisco-logo-color.svg)
    transaction/
      TransactionScreen.jsx/css  # Main POS screen — orchestrates everything
      TransactionTable.jsx/css   # Item list table with inline editing, search, barcode input
      TransactionControls.jsx/css # Sidebar controls (Transaction Type, Tax, Discount, Output)
      FunctionBar.jsx/css        # Active transaction function keys (F1-F12 row)
      IdleFunctionBar.jsx        # Idle state keys (Sale, Return, Reload Held, etc.)
    grid/
      ItemGrid.jsx/css       # Category grid → item sub-grid for restaurant menu
    customer/
      CustomerLookup.jsx/css # Customer search modal (name, phone, email)
    payment/
      PaymentModal.jsx/css   # Payment entry (cash/credit/debit/EBT, numpad, split payments)
      PaymentPreview.jsx     # Standalone page rendering PaymentModal with sample data (for Figma capture)
      HeldTransactionsModal.jsx/css # Reload or delete held transactions
    admin/
      AdminScreen.jsx/css    # Admin panel (settings, categories, items management)
```

### Routes
- `/` — Main POS transaction screen
- `/admin` — Admin panel (password protected)
- `/payment-preview` — Standalone payment modal with sample data (for Figma capture/dev)

### State Management (transactionStore.js)

Single Zustand store manages:
- `items[]` — current transaction line items
- `customer` — selected customer (or null)
- `transactionType` — sale | return | layaway | order | quote
- `taxType` — taxable | tax_exempt | alt_tax
- `discountMode` — none | by_line | all
- `outputType` — paper_tape | invoice
- `isActive` — whether a transaction is in progress
- `transactionNumber` — auto-incrementing (starts at 300001)
- `heldTransactions[]` — transactions put on hold
- Computed: `getSubtotal()`, `getTaxTotal()`, `getGrandTotal()`, `getEbtEligibleTotal()`, `getChange()`

### Two Function Bar States

1. **Idle** (`IdleFunctionBar`): Sale, Return, Lay-A-Way, Order, Quote, Alternate Tax Rate, Discount, Cash Check, Cash Drawer, Customer Inquiry, Open Drawer, Tax Type, Print Form, Payment, Pay Out, Reload Held Transaction, Item Inquiry, Exit
2. **Active** (`FunctionBar`): Repeat Last, Return Next, Quantity, Discount, Cancel, Coupon, Customer Inquiry, Write Memo, Put on Hold, Delete Last, Item Direct, Price, Sls. Change, Finish, Item Lookup, Item Inquiry, Tax Exempt Next, Open Drawer

### Keyboard Shortcuts
- **F1-F12** mapped to function bar actions (context-dependent on idle vs active)
- **Escape** cancels active transaction, returns to idle function bar
- **Enter** in code input submits barcode/quick code
- Payment modal: F1=Cash, F2=Debit, F3=Credit, F4=EBT

### Quick Codes (typed into item code input)
- `000` — Convenience Fee (taxable, price entered manually)
- `1` — Grocery (non-taxable, EBT eligible, price entered manually)
- `2` — Grocery Tax (taxable, price entered manually)
- `3` — Meat/Carne/Cheese (taxable, EBT eligible, price entered manually)
- `5` — Restaurant/Food (taxable, price entered manually)
- `11` — Boss Revolution (taxable, price entered manually)

### Item Search Priority
Search results are ordered: exact barcode match first, then barcode-starts-with, then partial name/barcode matches. Within each tier, results sort by **highest price first** so higher-priced items and duplicates with higher prices appear at the top.

## Design System (from Figma)

### Layout
- **Content wrapper:** `94.8vw` width, centered with `margin: 0 auto` (NO px max-width — always proportional to viewport)
- **Header:** Full-width, `#121212` background, 62px height, logo only (no store name text)
- **Sidebar:** 223px overlay panel, `position: fixed` starting at `top: 62px` (below header), overlays ALL content (table, totals, function keys). Slides in with `translateX` animation (280ms cubic-bezier). Always in DOM with `pointer-events: none/auto` toggle for smooth open/close transitions. Has close icon at top.
- **Table:** Split into fixed header (`.table-header-fixed`) + scrollable body (`.table-scroll`) so scrollbar only appears below the dark header row, never overlapping it. No border on `.table-outer`.
- **Search bar:** Absolutely centered in top bar with `position: absolute; left: 50%; transform: translateX(-50%)`, 320px width. Search dropdown `z-index: 200` renders above the table. No code hints in the top bar.
- **Totals bar:** Rounded 14px container, `align-items: center`, all total-fields 40px height with `justify-content: space-between` (except grand-total-field which is `height: auto`). Background: `rgba(235, 240, 241, 0.7)` with `backdrop-filter: blur(12px)`.
- **Function keys:** Rounded 14px container, two rows of 9 keys each, 85px key height. Background: `rgba(221, 227, 229, 0.7)` with `backdrop-filter: blur(12px)`.
- **Top bar:** Background: `rgba(237, 239, 241, 0.7)` with `backdrop-filter: blur(12px)`.
- **Bottom padding:** 35px below function keys
- **Empty table rows:** 50 rows to fill the scroll area on any screen

### Colors
- Header: `#121212`
- Active sidebar buttons: `#47ab77` (green) — NOT dark, green
- Inactive sidebar buttons: `#ffffff` bg, `#d1d9e6` border, `#4a5568` text
- Control group backgrounds: `#ecedf1`
- Sidebar background: `#fbfbfb`
- Customer section background: `#ecedf1`
- Customer lookup button: `#fbfbfb` bg, `#d1d9e6` border
- Item Grid button: `#47ab77`
- Scale button: `#ea8b0c`
- Admin button: `rgba(255,255,255,0.15)` with `rgba(255,255,255,0.3)` border
- Cancel key: white with `#e26666` border
- Finish key: `#47ab77` fill, `#c9f0dc` key text, `#f3f3f7` label text
- Put on Hold: plain white key (NOT orange highlighted)
- Table header: `#121212`
- Table row background: `rgba(251, 251, 251, 0.85)` (semi-transparent for gradient bleed)
- Row borders: `rgba(103, 133, 140, 0.3)`
- NT badge: `#e8ecf1` bg, `#1e40af` text
- Grand Total: `#282828` (38px bold)
- EBT Eligible: `#0d9488` (teal)
- Labels/muted text: `#8896a6`
- Primary text: `#1a1a2e`

### Payment Modal Colors
- Modal bg: `#ffffff`, rounded 12px, shadow `0 8px 32px rgba(0,0,0,0.15)`
- Pay input: `#eef2ff` bg, `#2563eb` 2px border (blue highlight)
- Due value: `#ca8a04` (gold/yellow)
- Change value: `#dc2626` (red)
- Cash button: `#2E7D32` (dark green)
- Credit Card button: `#1565C0` (blue)
- Debit Card button: `#6A1B9A` (purple)
- EBT SNAP button: `#E65100` (orange)
- Numpad keys: `#f2f3f5` bg, `#d1d8e0` border
- Quick amount keys ($1-$100): white bg, `#ca8a04` text
- Exact button: `#2E7D32` (green)
- Done button: `#2E7D32` (green)
- Cancel button: `#C62828` (red)
- Put on Hold button: `#f2f3f5` bg, `#d1d8e0` border
- Payments table header: `#121212` bg, white text
- SNAP Eligible: `#0d9488` border and value text

### Background Gradients
Three CSS `radial-gradient` on `#root` (NOT on body, NOT as DOM elements):
```css
background:
  radial-gradient(ellipse 80% 60% at 0% 0%, #b8c0c4 0%, rgba(184, 192, 196, 0) 100%),
  radial-gradient(ellipse 60% 50% at 100% 0%, #d4bc9a 0%, rgba(212, 188, 154, 0) 100%),
  radial-gradient(ellipse 90% 50% at 50% 100%, #7b8a8f 0%, rgba(123, 138, 143, 0) 100%),
  #e8e8e8;
```
Bottom gradient was darkened 20% from original `#9aacb3` to `#7b8a8f`.

**Critical:** Gradient end colors must fade to same-hue transparent (e.g. `rgba(184,192,196,0)`) NOT `transparent` (which is `rgba(0,0,0,0)` and causes muddy dark blending).

Content areas use semi-transparent backgrounds (70-85% opacity) with `backdrop-filter: blur(12px)` so gradients bleed through. The `.transaction-screen` background is `transparent`.

### Typography
- Font: Inter (loaded from Google Fonts, with system fallbacks)
- Table data: 14px
- Function key labels: 12px semibold
- Function key shortcuts: 9px bold, `#8896a6`
- Section labels: 10px bold uppercase, 0.5px tracking, `#8896a6`
- Control buttons: 10px semibold
- Button border-radius: 3px (sidebar controls), 10px (payment buttons), 8px (numpad)
- Grand Total: 38px bold `#282828`
- Payment modal title: 18px bold

### Sidebar Specs (from Figma node 6644:4954)
- Width: 223px
- Background: `#fbfbfb`
- Padding: 8px
- Gap between sections: 16px
- Gap between control groups: 14px
- Control group bg: `#ecedf1`, padding 6px, border-radius 6px
- Control button gap: 6px
- Shadow (Figma exact):
```css
box-shadow: 112px 0 31px rgba(0,0,0,0), 71px 0 29px rgba(0,0,0,0.01),
            40px 0 24px rgba(0,0,0,0.03), 18px 0 18px rgba(0,0,0,0.06),
            4px 0 10px rgba(0,0,0,0.06);
```
- Slide animation: `transform: translateX(-100%)` → `translateX(0)`, 280ms `cubic-bezier(0.25, 0.1, 0.25, 1)`

### Admin Screen Styling
- Admin tables have their OWN overrides — white `td` backgrounds, gray `th` headers (`#edeff1`), NOT the POS dark theme
- Categories table wrapped in `.inv-table-wrap` (same scrollable container as inventory maintenance)
- Admin header: `#121212` (matches POS header)
- `.inv-table-wrap`: white background, max-height 500px, overflow-y auto
- Login page: full-color-with-black-text logo (`jalisco-logo-color-blacktext.png`), green accent (`#47ab77`), green input focus ring

### Receipt Design (Thermal Printer)
Printed via `node-thermal-printer` (ESC/POS) on Epson/Star printers. Config in Admin → Settings.

**Layout (top to bottom):**
1. **Logo** — black logo image (`electron/assets/receipt-logo.png`) printed as bitmap. Falls back to bold text header if image fails.
2. **Store info** — name, address, phone (from `settings` table: `store_name`, `store_address`, `store_phone`)
3. **Transaction header** — date, time, receipt number (`R0403-300001` format = RMMDD-txnId), transaction #, customer name (if attached), transaction type (SALE/RETURN/etc.)
4. **Item table** — columns: QTY | ITEM | PRICE. Non-taxable items flagged with `[NT]`. Per-item discounts shown on indented line.
5. **Totals** — Subtotal, Tax, Discount (if any)
6. **Grand Total** — bold, larger text
7. **Payment** — payment type (CASH/CREDIT/DEBIT/EBT), amount paid, CHANGE DUE (bold)
8. **EBT section** — EBT Eligible total, EBT Applied amount (if EBT was used as payment)
9. **Footer** — bilingual "Thank you / Gracias", receipt reference number
10. **Auto-cut**

**Console fallback:** Same layout rendered as ASCII box art when no printer is connected (browser preview / development).

### Logo Variants
- **POS header** (dark bg): `public/jalisco-logo-color.svg` — colored hat, white text
- **Admin login** (light bg): `public/jalisco-logo-color-blacktext.png` — colored hat, black text
- **Receipt printing**: `electron/assets/receipt-logo.png` — all-black monochrome
- **SVG dark text**: `public/jalisco-logo-dark-text.svg` — colored hat, dark text (SVG)
- **Legacy**: `public/jalisco-tienda-mexicana-logo-white-rgb-2000px-w-72ppi.png` — all-white (unused)

Header shows logo only, no store name text.

**Branding source:** `/Documents/Claude/Projects/Tienda Mexicana Jalisco/Assets/Logo/` contains Full Color, Full Color with Black Text, White, and Black variants in Web (PNG/JPG/SVG) and Print (AI/EPS/PDF) formats.

## Important Rules

### CSS-Only UI Changes
When redesigning the UI, prefer CSS-only changes. Do NOT modify JSX unless the structure itself needs to change (like adding/removing DOM elements). All functionality is in the JSX/hooks/store — the CSS handles the visual design.

### No Tailwind
This project uses plain CSS with CSS variables. Do NOT install Tailwind or convert to Tailwind classes.

### Navigation
Uses BrowserRouter (not HashRouter). Navigation uses `window.location.href = '/path'` not `window.location.hash = '#/path'`.

### Z-Index Hierarchy
- `.sidebar-overlay`: z-index 50 (fixed, covers full viewport below header)
- `.left-panel-overlay`: z-index 51 (inside sidebar overlay)
- `.table-top-bar`: z-index 20 (relative, for search dropdown)
- `.search-wrap`: z-index 50 (absolute centered)
- `.search-dropdown`: z-index 200 (above table)
- `.modal-overlay` (globals): z-index 1000 (payment, customer, scale modals)
- **Important:** `.transaction-screen > *` rule sets `z-index: 1` on children — `.sidebar-overlay` and `.modal-overlay` are EXCLUDED from this rule so their z-index values work

### Mock API vs Electron API
- `window.api` is set by Electron's preload script in production
- `api-mock.js` installs a mock `window.api` for browser preview
- Always check `if (window.api)` before calling API methods
- Mock data includes categories with items, 3 sample customers, and 17K inventory items

### Barcode Scanner
USB barcode scanners send keystrokes rapidly (<50ms between chars) followed by Enter. The `useBarcodeScanner` hook detects this pattern and triggers item lookup. It only activates when focus is NOT on an input/textarea.

### Transaction Flow
1. User is in idle state (IdleFunctionBar shown)
2. Items added via: barcode scan, quick code, search, or item grid
3. Transaction auto-begins on first item (assigns transaction number)
4. Active FunctionBar replaces idle bar
5. User can: edit qty/price inline, remove items, apply discounts, hold transaction
6. ESC or F9 cancels → back to idle
7. F10 or "Finish" opens PaymentModal
8. Payment completed → receipt prints → transaction clears → back to idle

### Table Behavior
- Clicking anywhere on the table (not on inputs/buttons) dismisses active row editing and refocuses the hidden code input, showing the typing indicator cursor on the next empty row
- Table uses `scrollIntoView({ behavior: 'smooth', block: 'nearest' })` to avoid jumping when typing codes
- Table body scrolls independently from the fixed header
- `.table-outer` has NO border (removed — it was causing a visible white stroke)
- 50 empty rows fill the scroll area on any screen size

## Database (Electron only)

SQLite via `better-sqlite3`. Schema in `electron/database/schema.sql`.

### Tables
- **items** — barcode, name, price, category_id, button_color, is_taxable, is_ebt_eligible, grid_position, active
- **categories** — name, display_order
- **customers** — customer_number, name, phone, email
- **transactions** — timestamp (auto), customer_id, subtotal, tax_total, discount_total, grand_total, payment_type, amount_paid, change_given, transaction_type, ebt_amount, terminal_id, synced
- **transaction_items** — transaction_id, item_id, item_name, quantity, unit_price, line_total, discount
- **settings** — key/value pairs (tax_rate, store_name, store_address, store_phone, admin_password, printer_type, printer_interface, server_port, server_mode, ebt_enabled)

### Transaction Persistence
Every completed transaction is saved with all line items. The `dailySales` handler queries by date and returns: transaction count, total subtotal/tax/sales/discounts/EBT, breakdown by payment type, and full transaction list. Viewable in Admin → Sales tab.

### Backend Files
```
electron/
  main.js              # Electron main process
  preload.js           # Exposes window.api to renderer
  database/
    index.js           # SQLite connection setup
    schema.sql         # Table definitions
  ipc/
    handlers.js        # All IPC handlers (CRUD for items, categories, customers, transactions, settings, printer, network)
  services/
    printer.js         # Thermal receipt printing (ESC/POS)
    network.js         # Multi-terminal networking (primary/secondary)
  assets/
    receipt-logo.png   # Black logo for thermal printer
```

## Build & Run

```bash
npm run dev          # Vite dev server (browser preview with mock API)
npm run build        # Production build (dist/)
npm run dev:electron # Full Electron app with real database
```

### Figma Capture (HTML to Figma)
The `index.html` includes the Figma capture script (`https://mcp.figma.com/mcp/html-to-design/capture.js`). To use the capture toolbar:
1. Start dev server: `npx vite --port 3000`
2. Open in Chrome with capture hash params (generated by `generate_figma_design` tool)
3. Or use `/payment-preview` route for standalone payment modal capture
4. Requires the **HTML to Figma** Chrome extension installed
5. The capture toolbar appears when the URL contains `#figmacapture=...` hash params

## Deployment

GitHub Pages deployment serves the Vite build from `dist/` as a static site. The mock API (`api-mock.js`) provides all data for the browser preview. Cache-bust with `?v=N` query param.

## Known Patterns & Gotchas

- **Sidebar is always in the DOM** — uses `pointer-events: none/auto` + CSS transform for animation, NOT conditional rendering (`{sidebarOpen && ...}`)
- **`backdrop-filter` creates stacking contexts** — do NOT add it to `.table-scroll` or the search dropdown will render behind the table
- **`table-layout: fixed`** — do NOT use on the POS tables; it causes columns to not fill the container at narrow viewports
- **Gradient `transparent`** — never use bare `transparent` in gradients; it's `rgba(0,0,0,0)` and causes muddy transitions. Always use same-hue-transparent like `rgba(184,192,196,0)`
- **`position: fixed` on body** — can interfere with `background-image` rendering; gradients are on `#root` instead
- **Admin tables vs POS tables** — admin uses `.admin-content td { background: #ffffff }` to override the POS transparent row style
- **HashRouter vs BrowserRouter** — migrated to BrowserRouter with v7 future flags. Navigation uses `window.location.href` not `window.location.hash`. The `/payment-preview` route exists for Figma capture.
- **Figma capture + HashRouter conflict** — the `#figmacapture=...` hash fragment conflicts with HashRouter routes (causes blank page). This was a key reason for migrating to BrowserRouter.
- **Semi-transparent content areas** — top bar, totals bar, function bar, and table all use `rgba` backgrounds at 70-85% opacity with `backdrop-filter: blur(12px)` so the background gradients are visible through them. Do NOT make these opaque or the gradients disappear.

## Figma-to-Code Workflow

### HTML to Design Plugin (Local Dev → Figma)
The project uses the **HTML to Figma** plugin for bidirectional design-code sync:
1. Run local Vite dev server: `npx vite --port 3000`
2. The `index.html` loads the Figma capture script (`https://mcp.figma.com/mcp/html-to-design/capture.js`)
3. Open in Chrome with capture hash params (auto-generated by `generate_figma_design` tool)
4. The capture toolbar appears and lets you push live HTML frames directly into Figma
5. `/payment-preview` route renders the PaymentModal standalone for isolated capture
6. Requires the **HTML to Figma** Chrome extension installed in the browser

This workflow was used extensively during the April 3-4 session to validate CSS changes against the Figma source designs and push updated frames back into Figma for review.

## Changelog

### April 3-4, 2026 — Major UI Overhaul Session

Full POS redesign to match Figma designs (node-id=6637-3814 for main screen, node-id=6644-4602 for sidebar-open view, node-id=6644-4954 for sidebar detail).

**POS Home Screen:**
- Overhauled TransactionScreen layout to match Figma pixel-for-pixel
- Content wrapper changed to `94.8vw` centered (no fixed px max-width — always proportional)
- Background gradients fixed: three `radial-gradient` layers on `#root` with proper same-hue-transparent fade (not bare `transparent`). Bottom gradient darkened 20% from `#9aacb3` → `#7b8a8f`
- All content areas (top bar, totals, function keys, table) given semi-transparent `rgba` backgrounds (70-85% opacity) with `backdrop-filter: blur(12px)` for gradient bleed-through
- Search bar absolutely centered in top bar (`left: 50%; transform: translateX(-50%)`, 320px)
- Search dropdown z-index set to 200 (above table, below modals)
- Search results sorting: highest price first within each match tier
- Row divider color changed to `rgba(103, 133, 140, 0.3)` for subtlety
- Table rows given `rgba(251, 251, 251, 0.85)` semi-transparent background
- 50 empty rows added to fill scroll area on any screen
- Bottom padding set to 35px below function keys
- `.table-outer` border removed (was causing visible white stroke)

**Sidebar:**
- Refined to 223px width matching Figma node-id=6644-4954
- `position: fixed` starting at `top: 62px` (below header), overlays all content
- Slide animation: 280ms cubic-bezier, always in DOM with `pointer-events` toggle
- Multi-layer box-shadow from Figma (112px/71px/40px/18px/4px spread values)
- Control groups: `#ecedf1` background, 6px padding, 6px border-radius, 6px button gap
- Active buttons: `#47ab77` green; Inactive: white with `#d1d9e6` border
- Close icon added at top of sidebar

**Button Colors & Placement:**
- Item Grid button: `#47ab77` (green)
- Scale button: `#ea8b0c` (orange)
- Admin button: semi-transparent white with 30% border
- Cancel key: white with `#e26666` red border
- Finish key: `#47ab77` fill with light text
- Put on Hold: plain white (NOT orange)

**Admin Login Page:**
- Accent color changed to `#47ab77` green (input focus ring, button)
- Full-color-with-black-text Jalisco logo added (`jalisco-logo-color-blacktext.png`)

**Receipt Printing:**
- Thermal receipt design documented for ESC/POS printers (Epson/Star)
- Layout: logo → store info → transaction header → item table → totals → grand total → payment → EBT section → bilingual footer → auto-cut
- Receipt number format: `R0403-300001` (RMMDD-txnId)
- Console ASCII fallback for development/browser preview
- Three logo variants identified: color SVG (POS header), color-with-black-text PNG (admin), monochrome PNG (receipt)

**Backend:**
- Daily transaction tracking: `dailySales` IPC handler queries transactions by date, returns count, totals (subtotal/tax/sales/discounts/EBT), payment type breakdown, full transaction list
- Viewable in Admin → Sales tab

**Figma Integration:**
- HTML to Design plugin workflow established (local dev server → Figma capture)
- BrowserRouter migration completed (from HashRouter) to resolve `#figmacapture` hash conflict
- `/payment-preview` route added for standalone payment modal Figma capture
- Multiple Figma key frames documented for reference (main POS, sidebar open, sidebar detail, payment modal)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Graphicaljerry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
