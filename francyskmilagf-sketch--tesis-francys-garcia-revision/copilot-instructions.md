## tesis-francys-garcia-revision

> Laravel 12 educational platform with three roles: Profesor, Estudiante, Administrador.

# AGENTS.md - OVA Learning Platform

Laravel 12 educational platform with three roles: Profesor, Estudiante, Administrador.

## Tech Stack

- **Backend**: Laravel 12, PHP 8.4, Jetstream + Livewire
- **Frontend**: Tailwind CSS v3, Vite, jQuery, DataTables
- **Database**: MySQL 8 (production), SQLite in-memory (tests)
- **H5P**: `packages/h5p/h5p-php-library-master/` mapped as `H5P\` namespace

## Commands

```bash
composer install && npm install   # setup
composer run dev                  # starts: serve, queue, pail, vite (concurrently)
php artisan test                  # run all tests (SQLite in-memory)
php artisan test --filter=Name    # single test
php artisan pint                  # lint
npm run build                     # production build
```

## Frontend Quirks

- **jQuery**: aliased in `vite.config.js`, initialized in `resources/js/app.js`
- **DataTables**: import `datatables.net-dt` (NOT `jquery.dataTables`)
- **CSS**: `datatables.net-dt/css/dataTables.dataTables.min.css`

## Architecture

- Controllers: `app/Http/Controllers/{profesor,estudiante,administrador}/` (lowercase dirs)
- Views: `resources/views/{profesor,estudiante,admin}/`
- Activities: `resources/views/components/actividades/` (quiz, memoria, sopa-de-letras, dragdrop, crucigrama)
- OVA1 content: `resources/views/OVA1/`

## Testing

Tests use SQLite `:memory:` (see `phpunit.xml`). No MySQL needed for tests.

## Known Issues

- Controller namespaces use lowercase directories (non-PSR-4 style): `App\Http\Controllers\profesor\`
- `AppServiceProvider` is empty (boot/register do nothing)
- Route `/inicio` and `/` both render `inicio.blade.php` (view exists)

## Missing

- No CI workflows (`.github/workflows/`)
- No `.env.example` tracked (must create from `.env`)

## Common Pitfalls (learned the hard way)

- **Layout mismatch**: `layouts/app.blade.php` uses `{{ $slot }}` (Blade component syntax). Always use `<x-app-layout>` + content, NOT `@extends('layouts.app')` + `@section('content')`.
- **Guest pages crash**: `navigation-menu.blade.php` calls `Auth::user()->name` without `@auth` guard. Wrap user-dropdown blocks with `@auth` / `@endauth`.
- **MySQL connection errors**: Check `.env` has correct `DB_PORT` (commonly 3306, not 3307). If MySQL isn't running, switch to `DB_CONNECTION=sqlite` for dev.
- **Mail not sending**: `.env` uses `MAIL_MAILER=log` by default — emails go to `storage/logs/laravel.log`, not inbox. Use `bcrypt('password')` in tinker to reset passwords instead of email resets.

## UI Standardization (implemented)

- **Color primary**: `#6600FF` configured in `tailwind.config.js` as `primary` color scale (50-900)
- **Typography**: `Space Grotesk` (body), `Orbitron` (display/headings) via Google Fonts
- **Layout**: Dashboard admin style with fixed sidebar (`app.blade.php`), responsive mobile header
- **Components**: Reusable `<x-dashboard.card>`, `<x-dashboard.stat>`, `<x-sidebar-link>`
- **DataTables**: Styled header in `primary-600`, pagination with primary colors, initialized via `resources/js/app.js`
- **Forms**: Standardized inputs with `focus:ring-primary-500`, buttons: `btn-primary` (primary-600), `btn-secondary`, `btn-danger`, `btn-success`
- **Sidebar nav**: Role-based (`estudiante`/`profesor`/`administrador`) with active state highlighting

---
> Source: [francyskmilagf-sketch/TESIS-FRANCYS-GARCIA-REVISION](https://github.com/francyskmilagf-sketch/TESIS-FRANCYS-GARCIA-REVISION) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
