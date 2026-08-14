## caka

> This document provides the essential context needed to work effectively with this codebase. It is accurate as of the last time it was updated; when you change something described here, update this file too.

# Project Agent Guide

This document provides the essential context needed to work effectively with this codebase. It is accurate as of the last time it was updated; when you change something described here, update this file too.

---

## Project Overview

This is a **Laravel 13** web application skeleton. It is a brand-new, uncustomized Laravel project using modern Laravel conventions and the latest major versions of its toolchain. The application currently serves only the default Laravel welcome page at `/`.

- **Language**: PHP 8.3+
- **Framework**: Laravel 13.8
- **Frontend**: Vite 8 + Tailwind CSS v4 + Blade
- **Testing**: Pest PHP 4.7
- **Primary natural language**: English

---

## Technology Stack & Versions

### Backend
- `php`: ^8.3
- `laravel/framework`: ^13.8
- `laravel/tinker`: ^3.0

### Frontend
- `vite`: ^8.0.0
- `tailwindcss`: ^4.0.0
- `@tailwindcss/vite`: ^4.0.0
- `laravel-vite-plugin`: ^3.1
- `concurrently`: ^9.0.1

### Development & Testing
- `pestphp/pest`: ^4.7
- `pestphp/pest-plugin-laravel`: ^4.1
- `laravel/pint`: ^1.27 (code style)
- `laravel/pail`: ^1.2.5 (real-time log tailing)
- `laravel/pao`: ^1.0.6
- `nunomaduro/collision`: ^8.6
- `fakerphp/faker`: ^1.23
- `mockery/mockery`: ^1.6

---

## Directory Structure

Standard Laravel 13 layout:

```
app/
  Http/Controllers/     # HTTP controllers
  Models/               # Eloquent models
  Providers/            # Service providers
bootstrap/
  app.php               # Application bootstrap (Application::configure)
  providers.php         # Registered service providers
config/                 # Laravel configuration files
  app.php, auth.php, cache.php, database.php,
  filesystems.php, logging.php, mail.php, queue.php,
  services.php, session.php
database/
  factories/            # Eloquent model factories
  migrations/           # Schema migrations
  seeders/              # Database seeders
resources/
  css/app.css           # Tailwind CSS entry point
  js/app.js             # JavaScript entry point
  views/                # Blade templates
routes/
  web.php               # Web routes
  console.php           # Console / Artisan commands
tests/
  Feature/              # Feature tests (Pest)
  Unit/                 # Unit tests (Pest)
  Pest.php              # Pest configuration & shared helpers
  TestCase.php          # Base test case
public/                 # Web server document root
storage/                # Logs, caches, compiled views, etc.
```

---

## Build, Development & Test Commands

### Initial Setup
```bash
composer setup
```
This runs: `composer install`, copies `.env.example` to `.env`, generates `APP_KEY`, runs migrations, installs npm dependencies, and builds frontend assets.

### Development
```bash
composer dev
```
Runs four processes concurrently with color-coded output:
- `server` — `php artisan serve`
- `queue` — `php artisan queue:listen --tries=1 --timeout=0`
- `logs` — `php artisan pail --timeout=0`
- `vite` — `npm run dev`

If any process exits, all others are killed (`--kill-others`).

### Frontend Build
```bash
npm run dev     # Vite dev server with HMR
npm run build   # Production build (outputs to public/build/)
```

### Testing
```bash
composer test
```
This clears the config cache and then runs `php artisan test`, which delegates to Pest.

You can also run Pest directly:
```bash
./vendor/bin/pest
```

### Code Style
```bash
./vendor/bin/pint
```
Laravel Pint enforces the Laravel preset. Run this before committing.

### Common Artisan Commands
```bash
php artisan migrate:fresh --seed    # Reset DB and seed
php artisan db:seed                 # Run seeders
php artisan tinker                  # Interactive REPL
```

---

## Code Style Guidelines

- **PHP**: Follow the Laravel preset enforced by Pint. Key rules:
  - 4-space indentation
  - PSR-12 / Laravel coding style
  - Prefer explicit imports over facades when practical
- **EditorConfig** (already configured):
  - `charset = utf-8`
  - `end_of_line = lf`
  - `indent_size = 4` (2 for YAML files)
  - `insert_final_newline = true`
  - `trim_trailing_whitespace = true`
- **JavaScript / CSS**:
  - Vite + ES modules (`"type": "module"` in `package.json`)
  - Tailwind CSS v4 with `@import 'tailwindcss'` syntax
  - Font: Instrument Sans (400, 500, 600) loaded via Bunny Fonts through `laravel-vite-plugin/fonts`
- **Blade / HTML**: Use standard Blade directives. The welcome view uses Tailwind utility classes extensively.

---

## Testing Strategy

- **Framework**: Pest PHP (not plain PHPUnit)
- **Configuration**: `tests/Pest.php`
  - Feature tests extend `Tests\TestCase` via `pest()->extend(TestCase::class)->in('Feature')`
  - `RefreshDatabase` is available but **commented out** by default; uncomment if you need database isolation for Feature tests
- **Suites**:
  - `tests/Unit/` — Unit tests
  - `tests/Feature/` — HTTP / integration tests
- **Test Environment** (`phpunit.xml`):
  - `APP_ENV=testing`
  - `DB_CONNECTION=sqlite`
  - `DB_DATABASE=:memory:`
  - `CACHE_STORE=array`
  - `QUEUE_CONNECTION=sync`
  - `SESSION_DRIVER=array`
- **Helpers**: Add shared helpers to `tests/Pest.php` or create dedicated helper files.

---

## Runtime Architecture

- **Entry Point**: `public/index.php`
- **Routing**:
  - Web routes: `routes/web.php`
  - Console routes: `routes/console.php`
  - Health check: `/up`
  - API routes are not currently defined, but the exception handler is configured to render JSON for `api/*` requests
- **Middleware**: Configured in `bootstrap/app.php` via `withMiddleware()` — currently empty
- **Service Providers**: Registered in `bootstrap/providers.php` (only `AppServiceProvider` at the moment)
- **Queue**: Database-driven (`QUEUE_CONNECTION=database`)
- **Cache**: Database-driven (`CACHE_STORE=database`)
- **Session**: Database-driven (`SESSION_DRIVER=database`)
- **Broadcasting**: Log driver (`BROADCAST_CONNECTION=log`)
- **Mail**: Log driver (`MAIL_MAILER=log`)

---

## Database & Migrations

- **Default Connection**: MySQL (see `.env.example`)
- **Migrations** (default Laravel skeleton):
  - `0001_01_01_000000_create_users_table.php`
  - `0001_01_01_000001_create_cache_table.php`
  - `0001_01_01_000002_create_jobs_table.php`
- **Factories**: `database/factories/UserFactory.php`
- **Seeders**: `database/seeders/DatabaseSeeder.php` creates a single user (`Test User`, `test@example.com`)
- SQLite is used for tests only (`:memory:`).

---

## Frontend Asset Pipeline

- **Vite Config**: `vite.config.js`
  - Input files: `resources/css/app.css`, `resources/js/app.js`
  - Plugins: `laravel-vite-plugin`, `@tailwindcss/vite`
  - Fonts: `bunny('Instrument Sans', { weights: [400, 500, 600] })`
  - Watcher ignores `storage/framework/views/**`
- **CSS**: `resources/css/app.css` uses Tailwind CSS v4 `@import` and `@theme` syntax
- **JS**: `resources/js/app.js` is currently empty (no Alpine/Livewire yet)
- **Blade**: Views use `@vite(['resources/css/app.css', 'resources/js/app.js'])` and the `@fonts` directive
- The welcome page includes a fallback inline CSS block when Vite assets are not built (for first-run convenience)

---

## Security Considerations

- `APP_KEY` must be set (generated automatically by `key:generate`)
- `BCRYPT_ROUNDS=12` in `.env.example`
- `.npmrc` sets `ignore-scripts=true` to prevent arbitrary script execution during npm installs
- `.env` and `.env.*` are gitignored
- `storage/*.key` is gitignored
- No authentication scaffolding is installed yet. The welcome view conditionally shows login/register links only if `Route::has('login')` is true.

---

## Deployment Notes

- Ensure `APP_KEY` is set in production
- Run `npm run build` to compile frontend assets; `public/build/` should be present
- Run `php artisan migrate --force` for database migrations
- The application uses `optimize-autoloader` and `sort-packages` in Composer config
- Laravel assets are auto-published after `post-update-cmd`

---

## Useful References

- [Laravel 13 Documentation](https://laravel.com/docs)
- [Pest PHP Documentation](https://pestphp.com/)
- [Tailwind CSS v4 Documentation](https://tailwindcss.com/)
- [Vite Documentation](https://vitejs.dev/)

---
> Source: [antarsolusi/caka](https://github.com/antarsolusi/caka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
