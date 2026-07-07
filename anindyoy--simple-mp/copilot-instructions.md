## simple-mp

> This repository is a Laravel 12 + Filament 5 marketplace app with public storefront routes and an `/admin` Filament panel.

# AGENTS

This repository is a Laravel 12 + Filament 5 marketplace app with public storefront routes and an `/admin` Filament panel.

## Quick Start Commands

- Install and bootstrap: `composer run setup`
- Full local dev stack (server, queue listener, pail logs, vite): `composer run dev`
- Run tests: `composer test`
- Build frontend assets: `npm run build`

## Architecture Map

- Public web flows are in controllers and blade views. Start from [routes/web.php](routes/web.php).
- Admin/member panel behavior is configured in [app/Providers/Filament/AdminPanelProvider.php](app/Providers/Filament/AdminPanelProvider.php).
- Domain entities and relationships are in [app/Models](app/Models).
- Cross-entity business logic is in [app/Services](app/Services), especially product scheduling/moderation.
- Scheduled tasks are defined in [routes/console.php](routes/console.php).

## Project Conventions (Important)

- Many models use `protected $guarded = [];` (not explicit `$fillable`). Follow existing model style unless a task requires tightening assignment.
- Product visibility scheduling is cache-driven via `ProductScheduleService`; product create/delete hooks update schedule state.
- Product route model binding uses slug (`/product/{product:slug}`), not numeric id.
- Authorization is split across Laravel Policies and Filament resource `can*` methods.
- Email verification is required for authenticated panel access.

## High-Risk Pitfalls

- Do not change token refill scheduling behavior casually; daily/weekly refill commands are business-critical.
- Queue/scheduler assumptions matter: local dev uses queue listener in `composer run dev`; production still needs scheduler execution.
- Optional integrations (Turnstile, Telegram logging, SMTP) depend on env configuration and can fail silently if variables are missing.
- Filament admin plugin behavior changes by environment (local vs production), including developer login plugin and forced 2FA for admin in production.

## Validation Checklist Before Finishing

- Run `composer test` for backend-impacting changes.
- Run `npm run build` when touching frontend assets, Tailwind, or Vite configuration.
- Prefer minimal, targeted edits; preserve existing Indonesian-facing labels and user text unless asked otherwise.

## Reference Docs (Link, Do Not Duplicate)

- Product and feature overview: [README.md](README.md)
- User rules content: [peraturan-pengguna.md](peraturan-pengguna.md)
- Public routes and middleware constraints: [routes/web.php](routes/web.php)
- Scheduler definitions: [routes/console.php](routes/console.php)
- Filament panel setup and plugins: [app/Providers/Filament/AdminPanelProvider.php](app/Providers/Filament/AdminPanelProvider.php)

Refer to the coding guidelines located in .github/skills/ for all code generation.

---
> Source: [anindyoy/simple_mp](https://github.com/anindyoy/simple_mp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
