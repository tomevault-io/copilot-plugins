## rmc-mobile-apps

> > Mobile app for ordering construction vehicles, equipment, and readymix materials.

# RMC Readymix — CLAUDE.md

> Mobile app for ordering construction vehicles, equipment, and readymix materials.
> Full BRD: `DOCS/BRD/RMC_Readymix_BRD.docx`

---

## Quick Start

```bash
# Terminal 1 — Backend (MUST use Bun, not Node)
cd backend
bun run src/seed/seed.js     # first-time only: wipes + re-seeds DB
bun --watch src/index.js     # dev server with hot reload → http://localhost:3000

# Terminal 2 — Mobile app
cd mobile
npx expo start --ios         # iOS Simulator (localhost works natively)
```

**Test credentials**
| Role | Email | Password |
|------|-------|----------|
| Admin | `admin@rmc.com` | `admin123` |
| User | `user@rmc.com` | `user123` |
| User 2 | `rahul@builders.com` | `user123` |

---

## ⚠️ Critical: Runtime Constraint

**Node v24.11.1 is installed.** All native SQLite npm packages (`better-sqlite3`, `sqlite3`, etc.) fail to compile on Node v24 — no prebuilt binaries exist yet for this version.

**Fix:** The backend runs with **Bun v1.3.9**, which ships built-in `bun:sqlite` — a synchronous, file-persistent SQLite API nearly identical to `better-sqlite3`. All backend scripts must use `bun run` or `bun --watch`, never `node` or `nodemon`.

---

## Project Structure

```
rmc-mobile-app/
├── CLAUDE.md                              ← you are here
├── .gitignore                             # ignores node_modules, *.db, .expo/
├── DOCS/BRD/
│   └── RMC_Readymix_BRD.docx             # Full business requirements document
│
├── backend/                              # Express REST API — run with Bun
│   ├── package.json                      # all scripts use "bun run …"
│   ├── rmc.db                            # SQLite database file (gitignored)
│   └── src/
│       ├── index.js                      # Entry: mounts all routes, port 3000
│       ├── config/
│       │   ├── database.js               # bun:sqlite connection + initDB() (CREATE TABLE IF NOT EXISTS)
│       │   └── auth.js                   # JWT_SECRET, TOKEN_EXPIRY = '7d'
│       ├── middleware/
│       │   ├── auth.js                   # Verifies Bearer token → attaches req.user
│       │   ├── adminOnly.js              # Blocks non-admin roles with 403
│       │   └── errorHandler.js           # Global Express error handler
│       ├── models/                       # Raw SQL via db.prepare() — synchronous
│       │   ├── user.model.js             # findByEmail, findById, create, emailExists
│       │   ├── product.model.js          # findAll (filtered), findById, findFeatured,
│       │   │                             # getCategories, findAllAdmin, findByIdAdmin,
│       │   │                             # create, update, toggleActive
│       │   ├── cart.model.js             # getByUserId, findItem, addItem (upsert),
│       │   │                             # updateQuantity, removeItem, clearCart
│       │   └── order.model.js            # create, findByUserId, findById, findByIdAndUser,
│       │                                 # findAll (with customer JOIN), updateStatus,
│       │                                 # getNextInvoiceSequence
│       ├── controllers/
│       │   ├── auth.controller.js        # register, login, getProfile
│       │   ├── product.controller.js     # getProducts, getProductById, getCategories, getFeatured
│       │   ├── cart.controller.js        # getCart, addToCart, updateCartItem,
│       │   │                             # removeCartItem, clearCart
│       │   ├── order.controller.js       # checkout, getOrders, getOrderById
│       │   └── admin.controller.js       # getAllOrders, getOrderDetail, updateOrderStatus,
│       │                                 # getAllProducts, getProductDetail, createProduct,
│       │                                 # updateProduct, toggleProductActive
│       ├── routes/
│       │   ├── auth.routes.js            # /api/v1/auth/*
│       │   ├── product.routes.js         # /api/v1/products/*   (public)
│       │   ├── cart.routes.js            # /api/v1/cart/*       (user JWT required)
│       │   ├── order.routes.js           # /api/v1/orders/*     (user JWT required)
│       │   └── admin.routes.js           # /api/v1/admin/*      (admin JWT required)
│       ├── utils/
│       │   ├── invoice.js                # generateInvoiceNumber() → "RMC/YYYY-MM/XXXXX"
│       │   └── validators.js             # isValidEmail, isValidPassword, isValidPhone
│       └── seed/
│           └── seed.js                   # Wipes all data; inserts 3 users, 6 categories, 13 products
│
└── mobile/                               # React Native + Expo Router
    ├── package.json                      # main: "expo-router/entry"
    ├── app.json                          # name: "RMC Readymix", slug: "rmc-readymix", scheme: "rmc"
    ├── tsconfig.json                     # strict: true, paths: { "@/*": ["./*"] }
    ├── app/                              # File-based routes (Expo Router)
    │   ├── _layout.tsx                   # Root: wraps AuthProvider + CartProvider,
    │   │                                 # auth redirect via useSegments + useRouter
    │   ├── (auth)/
    │   │   ├── _layout.tsx               # Stack with saffron header
    │   │   ├── login.tsx                 # Email/password login + test credentials hint
    │   │   └── register.tsx              # Full registration form (name, email, phone, company, password)
    │   ├── (tabs)/                       # User role app
    │   │   ├── _layout.tsx               # Bottom tabs: Browse 🏠 | Cart 🛒 (badge) | My Orders 📋
    │   │   ├── (home)/
    │   │   │   ├── _layout.tsx           # Stack, saffron header
    │   │   │   ├── index.tsx             # Product list: search bar + category chips + FlatList grid
    │   │   │   └── product/
    │   │   │       └── [id].tsx          # Product detail: specs table, qty stepper, sticky Add to Cart
    │   │   ├── (cart)/
    │   │   │   ├── _layout.tsx           # Stack
    │   │   │   ├── index.tsx             # Cart: item rows with ± stepper, GST/delivery breakdown
    │   │   │   ├── checkout.tsx          # Address form, payment selector (COD default), notes
    │   │   │   └── confirmation.tsx      # Order confirmed: invoice number, total, CTA buttons
    │   │   └── (orders)/
    │   │       ├── _layout.tsx           # Stack
    │   │       ├── index.tsx             # Order history list (pull-to-refresh)
    │   │       └── [id].tsx              # Order detail: items, price breakdown, delivery address
    │   └── (admin)/                      # Admin role app
    │       ├── _layout.tsx               # Bottom tabs: Orders 📋 | Products 📦 | Dashboard 📊
    │       ├── dashboard.tsx             # KPI cards (total/pending/delivered/revenue) + logout
    │       ├── orders/
    │       │   ├── _layout.tsx           # Stack, dark header
    │       │   ├── index.tsx             # All orders + status filter chips (All/Pending/…)
    │       │   └── [id].tsx              # Order detail + status update buttons (transition-aware)
    │       └── products/
    │           ├── _layout.tsx           # Stack, dark header
    │           ├── index.tsx             # Product list (admin): all incl. inactive, Edit + Hide/Show per row
    │           ├── new.tsx               # Create product (uses ProductForm)
    │           └── [id]/
    │               └── edit.tsx          # Edit product pre-filled (uses ProductForm)
    ├── components/
    │   ├── rmc-button.tsx                # Variants: primary | outline | danger | ghost; loading spinner
    │   ├── rmc-input.tsx                 # Labeled TextInput with focus border + error message
    │   ├── status-badge.tsx              # Colored pill chip for order status
    │   ├── product-card.tsx              # Grid card: color avatar, availability badge, price, rating
    │   ├── order-card.tsx                # List card: invoice#, status badge, date, total; used by user + admin
    │   └── product-form.tsx              # Reusable form shared by new.tsx + edit.tsx:
    │                                     # chip selectors for category/price-unit/availability,
    │                                     # featured toggle, dynamic spec key-value editor
    ├── constants/
    │   └── theme.ts                      # COLORS, STATUS_COLORS, STATUS_LABELS, API_URL
    ├── contexts/
    │   ├── auth-context.tsx              # user, isLoading, login(), register(), logout()
    │   └── cart-context.tsx              # items, totals, itemCount, addToCart(), updateQuantity(),
    │                                     # removeItem(), refreshCart(), clearCart()
    └── services/
        └── api.ts                        # Axios instance: baseURL=localhost:3000/api/v1,
                                          # request interceptor injects Bearer token from AsyncStorage,
                                          # response interceptor shapes errors to Error.message
```

---

## Database Schema (SQLite — 5 tables)

```sql
-- All primary keys are UUID v4 strings

users
  id TEXT PK, full_name, email UNIQUE, phone, company_name,
  password_hash,                          -- bcrypt 12 rounds
  role TEXT DEFAULT 'user'                -- 'user' | 'admin' | 'superadmin'
  CHECK(role IN ('user','admin','superadmin')),
  is_verified INTEGER DEFAULT 1,          -- email verification (always 1 for MVP)
  is_active INTEGER DEFAULT 1,            -- soft-ban flag
  created_at, updated_at

categories
  id TEXT PK, name, description, is_active INTEGER DEFAULT 1

products
  id TEXT PK, name, category_id FK → categories,
  description TEXT,
  price REAL,
  price_unit TEXT DEFAULT 'per_unit'      -- 'per_m3' | 'per_day' | 'per_hour' | 'per_unit'
  CHECK(price_unit IN ('per_m3','per_day','per_hour','per_unit')),
  availability TEXT DEFAULT 'available'   -- 'available' | 'limited' | 'out_of_stock'
  CHECK(availability IN ('available','limited','out_of_stock')),
  specifications TEXT,                    -- JSON string, parsed on read
  image_url TEXT,                         -- reserved (no upload in MVP)
  is_featured INTEGER DEFAULT 0,          -- shown on home carousel
  is_active INTEGER DEFAULT 1,            -- soft-delete flag
  average_rating REAL DEFAULT 0,
  total_reviews INTEGER DEFAULT 0,
  created_at

cart_items
  id TEXT PK, user_id FK → users, product_id FK → products,
  quantity INTEGER DEFAULT 1,
  created_at,
  UNIQUE(user_id, product_id)             -- upsert increments quantity on conflict

orders
  id TEXT PK,
  invoice_number TEXT UNIQUE,             -- format: RMC/YYYY-MM/XXXXX (sequential per month)
  user_id FK → users,
  status TEXT DEFAULT 'pending'
  CHECK(status IN ('pending','confirmed','dispatched','in_transit','delivered','cancelled')),
  items TEXT NOT NULL,                    -- JSON snapshot of cart at checkout time
  subtotal REAL, gst_amount REAL,
  delivery_charge REAL DEFAULT 500,
  discount_amount REAL DEFAULT 0,
  grand_total REAL,
  payment_method TEXT DEFAULT 'cod',
  payment_status TEXT DEFAULT 'pending'
  CHECK(payment_status IN ('pending','paid','failed','refunded')),
  delivery_address TEXT,                  -- JSON: {address_line, city, state, pincode}
  notes TEXT,
  created_at, updated_at
```

---

## API Reference

**Base URL:** `http://localhost:3000/api/v1`

### Authentication (`/auth`)
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/register` | — | Register → `{ token, user }` |
| POST | `/auth/login` | — | Login → `{ token, user }` |
| GET | `/auth/profile` | User | Returns current user from token |

### Products (`/products`) — Public
| Method | Path | Query | Description |
|--------|------|-------|-------------|
| GET | `/products` | `?category_id=&search=` | Paginated product list (active only) |
| GET | `/products/categories` | — | All active categories |
| GET | `/products/featured` | — | Up to 5 featured products |
| GET | `/products/:id` | — | Single product; `specifications` parsed to object |

### Cart (`/cart`) — User JWT Required
| Method | Path | Body | Description |
|--------|------|------|-------------|
| GET | `/cart` | — | Items + computed totals |
| POST | `/cart` | `{ product_id, quantity }` | Add item (upserts; adds to existing qty) |
| PUT | `/cart/:id` | `{ quantity }` | Set quantity for cart item |
| DELETE | `/cart/:id` | — | Remove single item |
| DELETE | `/cart/all` | — | Clear entire cart |

### Orders (`/orders`) — User JWT Required
| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST | `/orders/checkout` | `{ delivery_address, payment_method, notes? }` | Create order from cart; clears cart on success |
| GET | `/orders` | — | Current user's order history |
| GET | `/orders/:id` | — | Single order (ownership-verified) |

### Admin (`/admin`) — Admin JWT Required
| Method | Path | Body | Description |
|--------|------|------|-------------|
| GET | `/admin/orders` | — | All orders with customer JOIN |
| GET | `/admin/orders/:id` | — | Full order detail |
| PUT | `/admin/orders/:id/status` | `{ status }` | Update status (transition-validated) |
| GET | `/admin/products` | `?search=` | All products including inactive |
| GET | `/admin/products/:id` | — | Single product (including inactive) |
| POST | `/admin/products` | `{ name, price, … }` | Create product |
| PUT | `/admin/products/:id` | any product fields | Update product (partial) |
| PATCH | `/admin/products/:id/toggle` | — | Flip `is_active` (soft-delete / restore) |

---

## Business Logic

### Checkout / Pricing
- **GST**: 18% of subtotal
- **Delivery**: ₹500 flat (MVP constant)
- **Invoice number**: `RMC/YYYY-MM/XXXXX` — sequential within current month, padded to 5 digits
- **Item snapshot**: `orders.items` stores a JSON copy of cart at checkout — price changes never affect past orders
- **Cart upsert**: `INSERT … ON CONFLICT(user_id, product_id) DO UPDATE SET quantity = quantity + excluded.quantity`
- **Cart clear**: happens server-side inside `checkout` controller after order creation

### Order Status Transitions
```
pending    → confirmed  | cancelled
confirmed  → dispatched | cancelled
dispatched → in_transit
in_transit → delivered
delivered  → (terminal — no further updates)
cancelled  → (terminal — no further updates)
```
Backend enforces this in `admin.controller.js → updateOrderStatus` via the `STATUS_TRANSITIONS` map.

### Auth / Navigation Flow (Mobile)
1. Root `_layout.tsx` reads token from AsyncStorage on mount → calls `GET /auth/profile`
2. `useEffect` watches `[user, isLoading, segments]` and calls `router.replace()`:
   - No user → `/(auth)/login`
   - `role === 'admin'` or `'superadmin'` → `/(admin)/orders`
   - Otherwise → `/(tabs)/(home)`
3. `CartContext` re-fetches from server whenever `user` changes (via `useCallback` + `useEffect`)

---

## Tech Stack

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| Mobile UI | React Native | 0.83.2 | Managed by Expo |
| App framework | Expo | SDK 55.0.6 | Managed workflow |
| Routing | Expo Router | 55.0.5 | File-based, `app/` directory |
| HTTP | Axios | ^1.13.6 | JWT interceptor in `services/api.ts` |
| Local storage | AsyncStorage | 2.2.0 | Token persistence |
| State | React Context | — | AuthContext + CartContext; no Redux |
| TypeScript | — | ~5.9.2 | strict mode, path alias `@/*` |
| Backend | Express | ^4.22.1 | REST API |
| Runtime | Bun | 1.3.9 | Required — built-in `bun:sqlite` |
| Database | SQLite (`bun:sqlite`) | — | Single file `backend/rmc.db` |
| Auth | jsonwebtoken + bcryptjs | 9.0.3 / 2.4.3 | 7-day JWT, bcrypt 12 rounds |
| IDs | uuid v4 | 9.0.1 | All primary keys |

---

## Brand / Design

All styling uses **inline style objects** — never `StyleSheet.create`. Use `borderCurve: 'continuous'` on all rounded corners.

| Token | Hex | Usage |
|-------|-----|-------|
| `COLORS.primary` | `#E8650A` | Saffron — buttons, headers, prices, CTAs |
| `COLORS.primaryLight` | `#FFF3EB` | Tinted background for info banners |
| `COLORS.primaryDark` | `#C4520A` | Pressed button state |
| `COLORS.dark` | `#1A1A1A` | Admin headers, splash |
| `COLORS.white` | `#FFFFFF` | Cards, backgrounds |
| `COLORS.lightGray` | `#F5F5F5` | Screen backgrounds |
| `COLORS.gray` | `#6B7280` | Secondary text, placeholders |
| `COLORS.border` | `#DDDDDD` | Input borders, dividers |
| `COLORS.success` | `#27AE60` | Delivered, available |
| `COLORS.warning` | `#F39C12` | Pending, limited stock |
| `COLORS.error` | `#E74C3C` | Cancelled, out of stock |
| `COLORS.info` | `#3B82F6` | Confirmed status |
| Purple | `#8B5CF6` | Dispatched status |

---

## Seed Data

```bash
cd backend && bun run src/seed/seed.js
```

Deletes all rows, then inserts:

| Type | Count | Details |
|------|-------|---------|
| Users | 3 | admin@rmc.com (admin), user@rmc.com (user), rahul@builders.com (user) |
| Categories | 6 | Readymix Concrete, Heavy Vehicles, Construction Equipment, Pumping Equipment, Transport Vehicles, Accessories & Tools |
| Products | 13 | 6 readymix grades (M20–M45), 2 transit mixers, 30t crane, 42m boom pump, tipper truck, vibrator, formwork panels |

All passwords hashed with bcrypt (12 rounds). Featured products: M20, M25, 8m³ Transit Mixer, 30t Crane, 42m Boom Pump.

---

## MVP Scope

### Built ✅
- Email/password auth (register + login) with 7-day JWT
- Role-based routing: user → `(tabs)`, admin → `(admin)`
- Product catalog: list with search + category filter, detail with spec table
- Shopping cart: add/update/remove, server-side persistence, totals (subtotal + 18% GST + ₹500 delivery)
- Checkout: address form, Cash on Delivery, order placement
- Order confirmation with invoice number
- User order history and detail view
- Admin: all orders list with status filter chips, order detail with status update buttons
- Admin: product management — list (incl. inactive), create, edit, soft-delete/restore

### Deferred (Phase 2+) ❌
| Feature | Reason |
|---------|--------|
| Razorpay payments | Needs sandbox keys; currently COD only |
| Google Maps GPS tracking | Needs API key + WebSocket driver |
| Push notifications (FCM) | Needs Firebase project setup |
| PDF invoice generation | PDFKit deferred; no download in MVP |
| Product image upload | S3 / CDN not configured |
| Social login (Google/Apple) | OAuth setup deferred |
| User reviews & ratings | Post-delivery only; needs delivered state |
| Redis caching | Not needed at SQLite scale |
| Wishlist / saved items | Phase 2 |

---

## Common Commands

```bash
# ── Backend ────────────────────────────────────────────────────────────
cd backend

bun run src/seed/seed.js              # Re-seed (destructive: wipes all data)
bun --watch src/index.js              # Dev server with hot reload
bun run src/index.js                  # Production start

# Quick API smoke-tests (backend must be running)
curl http://localhost:3000/health
curl http://localhost:3000/api/v1/products

TOKEN=$(curl -s -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@rmc.com","password":"user123"}' | bun -e \
  "const d = await Bun.stdin.text(); console.log(JSON.parse(d).token)")

curl http://localhost:3000/api/v1/cart -H "Authorization: Bearer $TOKEN"

# ── Mobile ─────────────────────────────────────────────────────────────
cd mobile

npx expo start --ios                  # iOS Simulator
npx expo start --android              # Android Emulator (change API_URL — see below)
npx tsc --noEmit                      # TypeScript type check (should be 0 errors)
```

---

## Android Development

For Android Emulator, update `mobile/constants/theme.ts`:

```ts
// iOS Simulator — use this
export const API_URL = 'http://localhost:3000/api/v1';

// Android Emulator — use this instead
export const API_URL = 'http://10.0.2.2:3000/api/v1';
```

---

## File & Code Conventions

| Convention | Rule |
|-----------|------|
| Route files | kebab-case `.tsx` inside `app/` |
| Component files | kebab-case (e.g. `product-card.tsx`) |
| Backend files | camelCase + type suffix (e.g. `auth.controller.js`) |
| Styling | Inline style objects only — no `StyleSheet.create` |
| Platform detection | `process.env.EXPO_OS` — not `Platform.OS` |
| AsyncStorage | `@react-native-async-storage/async-storage` — not from `react-native` |
| Context consumption | `useContext(Ctx)` — React 19 `React.use()` also works |
| Rounded corners | Always add `borderCurve: 'continuous'` alongside `borderRadius` |
| Safe area | `contentInsetAdjustmentBehavior="automatic"` on every `ScrollView` / `FlatList` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lalit2001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
