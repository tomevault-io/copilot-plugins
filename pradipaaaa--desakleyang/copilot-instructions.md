## desakleyang

> Laravel 13 + PHP 8.4 | PostgreSQL (Supabase, prod) / SQLite :memory: (test) | Tailwind CSS v4 + Vite | Node 22.x | Deployed on Railway

# AGENTS.md — Desa Kleyang

## Stack

Laravel 13 + PHP 8.4 | PostgreSQL (Supabase, prod) / SQLite :memory: (test) | Tailwind CSS v4 + Vite | Node 22.x | Deployed on Railway

## Quick start

```bash
composer run setup    # full setup: install, .env, key:generate, migrate, npm install & build
composer run dev      # server + queue + logs + Vite (concurrently)
composer run test     # config:clear && php artisan test
npm run build         # Vite production build
```

## Auth

Custom `AuthController` — not Breeze/Jetstream. Login at `/login`, seeded admin credentials: `admin@desa.com` / `password`.

## Routes

Public routes in `routes/web.php:14-26`, admin routes under `/admin` (`auth` middleware) at `routes/web.php:38-90`. Admin resource route parameters use Indonesian slugs (`perangkat-desa` → `official`, `struktur-organisasi` → `organization`, `kategori-berita` → `newsCategory`, etc.)

## App locale

Indonesian. Configured in `.env`: `APP_LOCALE=id`, `APP_FALLBACK_LOCALE=id`, `APP_FAKER_LOCALE=id_ID`. Validation messages in `resources/lang/id/`.

## Models

- Use `$fillable`, `$casts`, Eloquent scopes (`scopePublished`, `scopeUnread`)
- Slug auto-generated in `boot()` `creating` event via `Str::slug`
- `SoftDeletes` on `News`; `is_published` boolean on publishable models

## Admin CRUD pattern

Controllers use Form Request validation (`app/Http/Requests/`). File uploads stored via `Storage::disk('public')->store('images/{type}', 'public')`, old files deleted on update/destroy. Flash messages via `session('success')`.

## Views

Blade templates in `resources/views/frontend/` and `resources/views/admin/`. Two layouts: `layouts/app.blade.php` (public) and `layouts/admin.blade.php` (admin). Yields: `title`, `page_title`, `page_subtitle`, `content`, `head`, `scripts`.

## Testing

PHPUnit with in-memory SQLite (`phpunit.xml`). Suites: `tests/Unit/`, `tests/Feature/`. Run: `composer run test`.

## Deployment (Railway)

Defined in `railway.json`:
- Build: `npm run build`
- Deploy: `migrate --force`, `storage:link --force`, then `config:cache && route:cache && view:cache`

## Code style

- 4-space indent (`.editorconfig`)
- PHP 8.4 features (PHP attributes like `#[Fillable]` optional, mixed with traditional `$fillable`)
- Laravel Pint for formatting (`laravel/pint` in dev deps)

---
> Source: [pradipaaaa/DesaKleyang](https://github.com/pradipaaaa/DesaKleyang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
