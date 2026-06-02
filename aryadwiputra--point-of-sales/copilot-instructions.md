## point-of-sales

> - **Backend**: Laravel 12 (PHP 8.2+)

# AGENTS.md — Point of Sales

## Stack

- **Backend**: Laravel 12 (PHP 8.2+)
- **Frontend**: Inertia.js 2.0 + React 18, Vite 5
- **Styling**: Tailwind CSS 3 (custom theme in `tailwind.config.js`)
- **Auth/RBAC**: Spatie Laravel Permission + Laravel Breeze
- **DB**: MySQL (default); SQLite in-memory for tests
- **Payment webhooks**: Midtrans, Xendit (`routes/api.php`)

## Developer Commands

```bash
# Initial setup (run in order)
cp .env.example .env
composer install
npm install
php artisan key:generate
php artisan migrate --seed
php artisan storage:link

# Dev servers — run BOTH in parallel
npm run dev        # Vite (HMR)
php artisan serve  # Laravel API

# Testing
php artisan test                     # all tests
php artisan test --filter=FooTest    # single test class
php artisan test --filter=test_name  # single test method

# Formatting
vendor/bin/pint                      # PHP formatter (Laravel Pint)
```

## Architecture

- **Controllers**: `app/Http/Controllers/Apps/` — dashboard module controllers
- **React pages**: `resources/js/Pages/Dashboard/` — Inertia page components
- **Entry point**: `resources/js/app.jsx` — bootstraps Inertia + React
- **Services**: `app/Services/` — cross-cutting logic (AuditLog, CashierShift, StockMutation, Payments/)
- **Middleware**: `app/Http/Middleware/` — custom middleware including `EnsureActiveCashierShift`
- **Inertia shared props**: `app/Http/Middleware/HandleInertiaRequests.php` — shares auth, permissions, notifications, store profile to all pages
- **Routes**: `routes/web.php` (dashboard), `routes/api.php` (payment webhooks), `routes/auth.php` (Breeze auth)

## Middleware & Access Control

- **`permission`** (Spatie): every dashboard route is protected by a specific permission string
- **`active_shift`**: required for all POS transaction actions (search product, cart CRUD, hold/resume, store). Blocks if cashier has no open shift
- **`bot.guard`**: applied on login/register/forgot-password for throttle protection
- **`registration.enabled`**: public registration is **off by default** (`AUTH_PUBLIC_REGISTRATION=false`)

## Critical Gotchas

1. **Permission cache stale after seeding** — after `db:seed`, logout + login again to refresh Spatie's permission cache
2. **Webhooks need public APP_URL** — Midtrans/Xendit webhooks will not work with `localhost`; set `APP_URL` to a public URL
3. **Product images require storage link** — run `php artisan storage:link` or images won't render
4. **New module routes may 500 without migrations** — newer modules (sales returns, stock opname, cashier shifts, audit logs) need their migrations run: `php artisan migrate`
5. **Tests use SQLite in-memory** — `phpunit.xml` forces `DB_CONNECTION=sqlite`, `DB_DATABASE=:memory:`. Do not assume MySQL features in tests.
6. **Both dev servers required** — Inertia needs Vite running for HMR and asset serving. `php artisan serve` alone will not load JS/CSS.

## Frontend Conventions

- **Routing**: use Ziggy `route()` helper in React components (Ziggy is installed)
- **Styling**: Tailwind with semantic color tokens — `primary`, `accent`, `success`, `warning`, `danger` (see `tailwind.config.js`)
- **Icons**: `@tabler/icons-react`
- **Alerts**: `react-hot-toast` and `sweetalert2` (both installed)
- **Charts**: `chart.js`

## Database & Seeding

- Default seeders create roles, permissions, 2 users (admin + cashier), and sample data
- Admin: `arya@gmail.com` / `password`
- Cashir: `cashier@gmail.com` / `password`
- Indonesia region data loaded via `laravolt/indonesia` package

## Module Map (dashboard routes)

| Path | Purpose |
|------|---------|
| `/dashboard` | Main dashboard (permission: `dashboard-access`) |
| `/transactions` | POS — cart, hold/resume, checkout, print |
| `/transactions/history` | Transaction history + sales return entry |
| `/sales-returns` | Sales return management |
| `/customers` | Customer CRUD + Indonesia region data |
| `/receivables` | Customer receivables (piutang) |
| `/suppliers` | Supplier CRUD |
| `/payables` | Supplier payables (hutang) |
| `/stock-opnames` | Stock opname (audit inventory) |
| `/stock-mutations` | Stock mutation history |
| `/cashier-shifts` | Open/close cashier shifts |
| `/audit-logs` | Activity audit trail |
| `/settings/*` | Payments, bank accounts, store profile, targets |
| `/reports/sales` | Sales report |
| `/reports/profits` | Profit report |

## Docs

- Module docs: `docs/README.md` (index), `docs/features/` (per-module)
- Architecture: `docs/architecture-overview.md`
- Planning: `planning/improvement-planning.md`

---
> Source: [aryadwiputra/point-of-sales](https://github.com/aryadwiputra/point-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
