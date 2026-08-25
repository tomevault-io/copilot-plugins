## ecommercesite-1

> This file is the **single source of truth** for AI agents contributing to this project.

# NOVA Store — Agent Guide

This file is the **single source of truth** for AI agents contributing to this project.
Read it before making any changes. If anything is unclear, search the codebase before inventing conventions.

---

## 1. Project Overview

**NOVA Store** is a didactic fullstack e-commerce monorepo: a static HTML/CSS/JS frontend and an ASP.NET Core 8 Web API backed by SQL Server.

```
Browser (static HTML, port 5500)
       |
   fetch()  ←───  REST API (http://localhost:5036)
       |
  ASP.NET Core 8
       |
  Entity Framework Core + ASP.NET Identity
       |
  SQL Server (NovaStoreDb)
```

| Layer | Technology | Notes |
|-------|-----------|-------|
| Frontend | HTML5, CSS3, Vanilla JS | No build step. Multi-page app (not SPA) |
| Frontend libs | Bootstrap 5.3.3 CSS, Bootstrap Icons 1.11.3, Google Fonts "Inter" | Bootstrap grid/utilities; Bootstrap JS only where pages already load it |
| Backend | ASP.NET Core 8, C# 12 | `.NET 8.0` SDK (`global.json`) |
| ORM | Entity Framework Core 8 | Code-first, Fluent API, manual DTO mapping (no AutoMapper) |
| Auth | ASP.NET Identity + JWT Bearer | Roles: `Admin`. OAuth: Google + Microsoft |
| Payments | Stripe Checkout | `PaymentsController`, webhook optional locally |
| Email | SMTP via `EmailService` | Mailtrap in dev; empty `SmtpHost` logs reset link to console |
| Database | SQL Server | Connection string in `appsettings.json` |
| API docs | Swagger / Swashbuckle | `/swagger` in Development |

### Implemented features (work packages)

| WP | Feature | Status |
|----|---------|--------|
| Core | Products, categories, cart, checkout, orders | Done |
| Auth | Register, login, profile, JWT | Done |
| OAuth | Google login, Microsoft login | Done |
| Payments | Stripe Checkout + order verify | Done |
| WP1 | Forgot/reset password email | Done |
| WP2 | Product search (`?search=`, header search) | Done |
| WP3 | Admin panel (roles, CRUD, dashboard) | Done |
| WP4 | Transactional emails (order confirmation, welcome) | Done |
| Post-WP4 | Auth-required checkout + order history (`orders.html`) | Done |

### Team workflow

- **OpenCode (DeepSeek)** — plans features and writes execution specs
- **Cursor** — implements code following this guide
- Work package order: **WP1 → WP2 → WP3 → WP4**

---

## 2. Dev Environment

### Quick start (Windows)

```bat
start-dev.bat
```

Opens two CMD windows:
- Backend: `http://localhost:5036` (Swagger at `/swagger`)
- Frontend: `http://localhost:5500`

Scripts live in `scripts/` (`start-backend.bat`, `start-frontend.bat`). Use **CMD**, not PowerShell, when paths contain spaces (`Ecommerce project - 1`).

### Manual start

```bash
# Backend
cd backend/NovaStore.Api
dotnet ef database update    # first time / after new migration
dotnet run --launch-profile http

# Frontend
cd frontend
python -m http.server 5500
```

### URLs & config

| What | Value |
|------|-------|
| API base (frontend) | `http://localhost:5036/api` — defined in `frontend/assets/js/api.js` as `API_BASE` |
| Frontend | `http://localhost:5500` |
| CORS policy name | `FrontendCors` — allows all origins in Development |
| Admin seed user | `admin@novastore.hr` / `Admin123!` (from `Admin` section in appsettings) |

**Secrets:** `appsettings.json` contains dev keys (Stripe, Mailtrap, JWT). Do not commit production secrets. Do not paste real secrets into chat or docs.

---

## 3. Project Structure

```
Ecommerce project - 1/
├── AGENTS.md                  # ← you are here
├── README.md
├── start-dev.bat
├── start-dev.ps1
├── scripts/
│   ├── start-backend.bat
│   └── start-frontend.bat
│
├── frontend/                  # STATIC CLIENT (no build)
│   ├── index.html
│   ├── shop.html              # filters, sort, ?search=
│   ├── product.html
│   ├── cart.html              # localStorage
│   ├── checkout.html          # Stripe checkout
│   ├── order-success.html
│   ├── categories.html
│   ├── about.html
│   ├── contact.html
│   ├── login.html             # email + Google + Microsoft
│   ├── register.html
│   ├── profile.html
│   ├── orders.html            # order history (requires auth)
│   ├── forgot-password.html
│   ├── reset-password.html
│   ├── admin.html             # dashboard
│   ├── admin-products.html
│   ├── admin-orders.html
│   ├── admin-categories.html
│   ├── admin-users.html
│   └── assets/
│       ├── css/style.css      # single stylesheet (design system + admin)
│       └── js/
│           ├── products.js    # mock data + getters (fallback)
│           ├── api.js         # API_BASE, fetchProducts, fetchMyOrders(), loadStoreData()
│           ├── auth.js        # JWT, authFetch, isAdmin(), requireAuth(), getLoginRedirectUrl()
│           ├── cart.js        # localStorage cart, formatPrice()
│           ├── main.js        # header/footer, NAV_LINKS, productCardHTML
│           ├── admin.js       # requireAdmin(), adminFetch(), admin layout
│           └── msal-browser.min.js   # Microsoft login (local, not CDN)
│
└── backend/
    ├── global.json
    ├── NovaStore.sln
    └── NovaStore.Api/
        ├── Program.cs         # DI, CORS, JWT, Identity, Stripe, AdminSeeder
        ├── appsettings.json
        ├── appsettings.Development.json
        ├── Controllers/
        │   ├── ProductsController.cs
        │   ├── CategoriesController.cs
        │   ├── OrdersController.cs
        │   ├── AuthController.cs
        │   ├── PaymentsController.cs
        │   └── AdminController.cs    # [Authorize(Roles = "Admin")]
        ├── Models/
        │   ├── Entities/
        │   └── DTOs/
        ├── Data/
        │   ├── AppDbContext.cs
        │   ├── AdminSeeder.cs
        │   └── Migrations/           # see §6
        └── Services/
            ├── TokenService.cs       # JWT + role claims
            └── EmailService.cs
```

---

## 4. Code Conventions

### 4.1 General

| Rule | Standard |
|------|----------|
| Indentation | 2 spaces (frontend), 4 spaces (C#) |
| Encoding | UTF-8 |
| UI text, code comments, commits | **Croatian** (this file stays English) |
| Scope | Minimal diffs; match existing patterns; no over-engineering |

### 4.2 Frontend — HTML

- `<html lang="hr">`, double-quoted attributes, 2-space indent
- `data-page="..."` on `<body>` — nav highlight via `NAV_LINKS` in `main.js` (admin pages use `data-page="admin"`)
- Header/footer: `<div id="nova-header">` / `<div id="nova-footer">` (injected by `main.js`)
- **Standard script order** (shop pages with cart):

  1. `bootstrap.bundle.min.js` (if page uses Bootstrap JS)
  2. `products.js`
  3. `api.js`
  4. `auth.js`
  5. `cart.js`
  6. `main.js`
  7. page-specific inline `<script>`

- **Auth-only pages** (login, profile): skip `products.js` / `api.js` / `cart.js` where not needed
- **Checkout / orders pages**: `auth.js` → `api.js` (if needed) → `cart.js` → `main.js` → inline script
- **Admin pages**: `auth.js` → `main.js` → `admin.js` → inline script. Content goes in `<div id="admin-root">`

### 4.3 Frontend — CSS

- Single file: `frontend/assets/css/style.css`
- Numbered section comments: `/* ---------- N. Section name ---------- */`
- Design tokens in `:root`; custom classes prefixed `nova-` (`.btn-nova`, `.form-control-nova`, `.container-nova`)
- Admin styles in section **18. Admin panel**; order history uses `.order-card`, `.order-item-row` (same file)
- Breakpoints: 992px, 768px, 540px

### 4.4 Frontend — JavaScript

- No modules, no bundler — globals via script load order
- `SCREAMING_SNAKE_CASE` constants, `camelCase` functions
- `async/await` for API; errors via `parseAuthError()` pattern in `auth.js`
- Prices: `formatPrice()` from `cart.js` (or `adminFormatPrice()` in `admin.js`)
- Toasts: `showToast()` from `main.js`

### 4.5 Backend — C#

- File-scoped namespaces, nullable enabled, Allman braces
- `/// <summary>` on public API surface
- DTO suffix: `Dto`, `CreateXDto`, `UpdateXDto`
- Controllers: `[ApiController]`, `ControllerBase`, private `MapToDto()` / `MapX()` — **no AutoMapper**
- Admin routes: `[Route("api/admin")]`, `[Authorize(Roles = "Admin")]`
- Public routes: `[Route("api/[controller]")]`
- Never trust client prices — read from DB in `OrdersController`

### 4.6 Database / EF Core

- `AppDbContext : IdentityDbContext<ApplicationUser>`
- Delete: `Restrict` for Product↔OrderItem, Category↔Product; `Cascade` for owned children
- `Order` → `ApplicationUser`: `SetNull` on user delete (historical orders kept)
- `Order.Status` stored as string enum
- Seed via `HasData()` — IDs: categories **1–7**, products **1–12**, images `productId*10 + n`
- New seed IDs must not collide

---

## 5. Entity & DTO Reference

### Entities (highlights)

| Entity | Notes |
|--------|-------|
| **Product** | CategoryId, Price, OldPrice, Rating, Reviews, Badge, InStock, Images collection |
| **Category** | Name (unique), Icon, ImageUrl |
| **Order** | `UserId` (FK, nullable for legacy rows), customer fields copied at checkout, Stripe IDs |
| **OrderItem** | ProductName + UnitPrice **snapshots** |
| **ApplicationUser** | FullName, GoogleId, MicrosoftId, CreatedAt; `Orders` collection |

### DTOs (additions beyond basics)

| DTO | Notes |
|-----|-------|
| `AuthResponseDto` | Token, ExpiresAt, Email, FullName, **Roles** |
| `UserProfileDto` | … + **Roles** |
| `DashboardDto` | ProductCount, OrderCount, TotalRevenue, … |
| `AdminProductDto` | Includes CategoryId, CreatedAt |
| `CreateProductDto` / `UpdateProductDto` | Images as `List<string>` URLs |
| `AdminUserDto` | HasGoogle, HasMicrosoft, Roles |
| `ForgotPasswordDto`, `ResetPasswordDto` | WP1 password reset |

---

## 6. Migrations (applied)

| Migration | Purpose |
|-----------|---------|
| `InitialCreate` | Categories, products, images |
| `AddOrders` | Orders + order items |
| `AddIdentity` | ASP.NET Identity tables |
| `AddGoogleId` | OAuth Google |
| `AddMicrosoftId` | OAuth Microsoft |
| `AddStripePaymentFields` | Stripe IDs on Order |
| `AddOrderUserId` | `Order.UserId` FK → `AspNetUsers` |

Create new: `dotnet ef migrations add <Name>` then `dotnet ef database update`.

WP3 admin uses Identity roles — **no extra migration** for admin (roles in `AspNetRoles`).

---

## 7. API Reference

### Public store (read-only)

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| GET | `/api/products?category=&sort=&search=` | No | Product list |
| GET | `/api/products/{id}` | No | Product detail |
| GET | `/api/categories` | No | Categories + counts |

### Orders (authenticated customer)

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/orders` | Yes | Create order; sets `UserId` + email from account |
| GET | `/api/orders/mine` | Yes | Current user's orders (newest first) |
| GET | `/api/orders/{id}` | Yes | Order detail — **own orders only** (`Forbid` otherwise) |

### Auth

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/auth/register` | No | Register |
| POST | `/api/auth/login` | No | Login → JWT + roles |
| POST | `/api/auth/google-login` | No | Google ID token |
| POST | `/api/auth/microsoft-login` | No | Microsoft ID token |
| GET | `/api/auth/me` | Yes | Profile + roles |
| PUT | `/api/auth/profile` | Yes | Update name, phone |
| POST | `/api/auth/change-password` | Yes | Change password |
| POST | `/api/auth/forgot-password` | No | Send reset email |
| POST | `/api/auth/reset-password` | No | Set new password |

### Payments (Stripe)

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/payments/create-checkout-session` | Yes | Stripe session — **order must belong to caller** |
| POST | `/api/payments/verify-session` | No | Confirm payment, set Paid, send confirmation email |
| POST | `/api/payments/webhook` | Stripe sig | Optional locally |

### Admin (`[Authorize(Roles = "Admin")]`)

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/admin/dashboard` | Stats |
| GET/POST/PUT/DELETE | `/api/admin/products` | Product CRUD |
| GET/POST/PUT/DELETE | `/api/admin/categories` | Category CRUD |
| GET | `/api/admin/orders?status=` | All orders |
| PUT | `/api/admin/orders/{id}/status` | Update status |
| GET | `/api/admin/users` | Users (read-only) |

**Error shapes:** `{ message: "..." }` or `{ errors: ["..."] }`.

---

## 8. Key Patterns

### Mock → API fallback

`products.js` defines `NOVA_PRODUCTS` / `NOVA_CATEGORIES`. `api.js::loadStoreData()` fetches API data and **mutates arrays in place** (never reassign). On failure, mock data remains. Check `window.__NOVA_DATA_SOURCE`.

### Auth & roles

1. Login/register/OAuth → JWT with `ClaimTypes.Role` claims (`TokenService`)
2. `localStorage`: `nova_auth_token`, `nova_auth_user` (includes `roles`)
3. `authFetch()` adds `Authorization: Bearer`
4. `isAdmin()` — `roles.includes("Admin")`
5. `requireAdmin()` in `admin.js` — calls `/api/auth/me`, redirects if not admin
6. Admin link in navbar only when `isAdmin()` (`main.js`)
7. Re-login required after role seed if session predates admin setup

### Checkout & order history (auth-required)

1. **Cart** (`cart.html`) — guest can browse/add; `checkout()` redirects to `login.html?redirect=checkout.html` if not logged in
2. **Checkout** (`checkout.html`) — `requireAuth()` on load; uses `authFetch()` for `POST /api/orders` and `create-checkout-session`
3. Email field on checkout is **read-only** (taken from account); name/phone prefilled via `fetchProfile()`
4. **Order history** (`orders.html`) — `fetchMyOrders()` → `GET /api/orders/mine`; link in profile + mobile drawer
5. **Login redirect** — `getLoginRedirectUrl()` + `?redirect=` query param (login ↔ register preserve redirect)
6. `requireAuth()` appends current page as `redirect` when sending user to login

### Admin product delete

`OrderItem` → `Product` is **Restrict**. Deleting a product in orders returns **409** — use `InStock = false` instead.

### Order snapshots & ownership

- `OrderItem.ProductName` and `UnitPrice` copied at creation (historical accuracy)
- `Order.UserId` set on create from JWT; delivery fields still stored on `Order`
- Legacy orders with `UserId = null` do not appear in `/api/orders/mine`
- Admin sees all orders via `/api/admin/orders` (not customer endpoint)

### Search (WP2)

- API: `GET /api/products?search=term`
- Frontend: header form in `main.js`, `shop.html?search=`, `fetchProducts()` in `api.js`

### Email (WP1 + WP4)

- `EmailService` + `Email:*` config
- If `SmtpHost` empty, email body logged to backend console (dev fallback)
- `SendPasswordResetAsync`, `SendWelcomeAsync`, `SendOrderConfirmationAsync`

### Stripe

- Checkout from `checkout.html` → `order-success.html` with `verify-session`
- Webhook optional for local dev

---

## 9. Agent Instructions

### 9.1 Before any task

1. Read this file and grep the codebase for similar implementations
2. Identify affected layer(s): frontend, backend, migrations, config
3. Keep changes minimal; do not refactor unrelated code
4. UI strings and comments in Croatian
5. Do not commit unless the user asks

### 9.2 Frontend agent

**Scope:** `frontend/**`

- New shop pages: copy existing HTML, set `data-page`, use standard script order
- Product cards: always `productCardHTML(p)` from `main.js`
- New admin UI: use `admin.js` helpers (`renderAdminLayout`, `adminFetch`, `requireAdmin`)
- Adding nav item: update `NAV_LINKS` in `main.js`
- CSS: add to the correct numbered section in `style.css`
- Protected pages: `requireAuth()` (customer) or `requireAdmin()` (admin)
- Checkout/order flows: always `authFetch()`, never plain `fetch()` for order/payment endpoints

### 9.3 Backend agent

**Scope:** `backend/NovaStore.Api/**`

- New public endpoint → appropriate `*Controller.cs`
- Admin features → `AdminController` or new controller with `[Authorize(Roles = "Admin")]`
- New entity: Entity → DbContext → migration → DTO → controller mapping
- JWT changes → `TokenService` + `Program.cs` `RoleClaimType`
- Startup seed → `AdminSeeder` or dedicated seeder called from `Program.cs`
- Order endpoints: `[Authorize]` on `OrdersController`; verify `UserId` ownership before returning data
- Password rules relaxed for dev (min 6 chars)

### 9.4 Database agent

**Scope:** `Data/Migrations/`, `AppDbContext`, seed data

- Never edit applied migrations — add new ones
- Verify delete behaviors and decimal precision
- Coordinate seed ID ranges with backend agent

### 9.5 Transactional emails (WP4 — implemented)

- **Welcome:** `SendWelcomeAsync()` after new user in `Register`, `GoogleLogin`, `MicrosoftLogin`
- **Order confirmation:** `SendOrderConfirmationAsync()` after payment in `VerifySession` and `Webhook`
- Order must include `.Include(o => o.Items)` before sending confirmation
- Email only sent on Pending → Paid transition (avoids duplicates on success page refresh)
- Reuse `EmailService`; HTML templates inline in the service

---

## 10. Collaboration Workflow

```
Feature request (OpenCode plan)
    │
    ├── Backend: entity/DTO/controller/migration/seed
    ├── Database: review migration, indexes, seed IDs
    └── Frontend: HTML + JS + CSS wired to real API
```

**Handoffs:**
- Backend defines API contract first
- Frontend `ProductDto` shape = `products.js` mock shape = API response
- Admin features need both `AdminController` and `admin-*.html` pages

---

## 11. Do Not Break

1. **Script load order** — globals depend on it; insert new files at the correct position
2. **Array references** — `NOVA_PRODUCTS` / `NOVA_CATEGORIES` mutated in place, not reassigned
3. **Client prices** — never send prices in `CreateOrderDto`
4. **Seed IDs** — categories 1–7, products 1–12; avoid collisions
5. **CORS** — keep `FrontendCors` working for `localhost:5500`
6. **Delete behavior** — Product in orders cannot be hard-deleted
7. **Admin security** — protect on server (`[Authorize(Roles = "Admin")]`), not only hide UI links
8. **JWT roles** — required for admin; `RoleClaimType` must stay configured
9. **Order auth** — do not make `POST /api/orders` or `create-checkout-session` public again
10. **Connection string** — user-specific SQL Server instance; do not hardcode in code
11. **Path with spaces** — use `start-dev.bat` / CMD, or quote paths in scripts
12. **API file lock** — stop running `NovaStore.Api.exe` before `dotnet build` if copy fails

---

## 12. Known Quirks

| Topic | Note |
|-------|------|
| `sw.js` 404 | Not part of project; ignore in Python server log |
| Microsoft OAuth | Azure app registration may need correct account types / manifest |
| Stripe webhook | Optional locally; `verify-session` works without Stripe CLI |
| Old JWT sessions | Lack `roles` until re-login or `refreshAuthRolesIfNeeded()` in `main.js` |
| Legacy orders | `UserId = null` — not shown in `orders.html` / `/api/orders/mine` |
| `frontend/assets/js/Untitled` | Stray file; safe to ignore or delete if cleaning up |

---

*Last updated: auth-required checkout, order history (`Order.UserId`, `orders.html`).*

---
> Source: [Jure135/EcommerceSite-1](https://github.com/Jure135/EcommerceSite-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
