## financely

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
composer run dev    # Start dev server, queue listener, and Vite watcher concurrently
npm run build       # Production asset build
```

### Testing
```bash
composer test       # Clears config cache, then runs full test suite via Pest PHP
php artisan test --filter=<TestName>   # Run a single test or test class
```

### Code Style
```bash
./vendor/bin/pint   # Format PHP code (Laravel Pint)
```

## Architecture

**Laravel 12 + Livewire 3** application — no separate API or SPA. All interactivity is handled server-side via Livewire components. There is only one traditional HTTP controller (`Auth/VerifyEmailController`); everything else is Livewire.

### Core Models (all use UUID primary keys, all scoped to `User`)
- **Transaction** — Income or expense record; type enum `income|expense`, state enum `paid|pending`; optional `expected_payment_date` for pending payments; has accessor `getIsPaymentFutureAttribute()` used for due-date highlighting
- **Category** — User-defined category with auto-generated slug; `is_active` boolean; cannot be deleted if transactions exist (enforced via DB constraint)
- **Obligation** — Recurring monthly billing with `limit_day`; can auto-generate transactions; toggleable active/inactive

### Authorization
All three models have Policies (`CategoryPolicy`, `TransactionPolicy`, `ObligationPolicy`) that enforce `user_id` ownership for `update` and `delete` operations. Data isolation is tested in `tests/Feature/*/` `*AuthorizationTest` and `*DataIsolationTest` files.

### Livewire Component Pattern
Components live under `app/Livewire/<Feature>/`. The pattern varies by feature:
- **Transactions**: `Create`, `Update`, `Index`, `Filters`, `PendingTransactions`
- **Categories**: flat naming — `CategoryCreate`, `CategoryEdit`, `CategoryIndex` (no subfolder nesting)
- **Obligations**: `MonthlyBillings` (single component)
- **Metrics**: `Grid` (dashboard)
- **Settings**: `Profile`, `Password`, `Appearance`, `DeleteUserForm`

Form objects in `app/Livewire/Forms/` (`TransactionForm`, `CategoryForm`, `ObligationForm`) use `#[Validate]` attributes and handle both `store()` and `update()` logic.

### Feature Areas
- **Transactions** (`/transaction`, `/transaction/{id}/edit`) — Create, edit, list by month, filter, view pending payments with due-date highlighting (`/pending-transactions`)
- **Categories** (`/category`, `/category/{id}/edit`) — CRUD with slug management and active/inactive toggle
- **Obligations** (`/obligation`) — Monthly billings that can spawn transactions
- **Dashboard** (`/dashboard`) — Metrics grid; requires `auth` + `verified` middleware
- **Settings** (`/settings/*`) — Profile, password, appearance, account deletion
- **Auth** (`routes/auth.php`) — Livewire-based login, register, password reset, email verification

### Frontend
- **Blade templates** in `resources/views/`, with `layouts/app.blade.php` as the main shell
- **Flux** UI component library (pre-built components accessed as `<flux:*>`)
- **Tailwind CSS v4** via Vite plugin
- **flatpickr** for date pickers
- **PWA** support via `silviolleite/laravelpwa` (installable, offline-capable)
- **Livewire Volt** available for single-file components (`livewire/volt`)

### Testing Setup
- Uses **Pest PHP v3**
- In-memory SQLite database (configured in `phpunit.xml`)
- Feature tests live in `tests/Feature/`, unit tests in `tests/Unit/`
- Model factories available for all core models; `TransactionFactory` has a `.pending()` state
- Test focus areas: authorization/policy enforcement, data isolation between users, Livewire component interactions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasanzdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
