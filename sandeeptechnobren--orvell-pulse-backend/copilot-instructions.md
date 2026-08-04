## orvell-pulse-backend

> Stateless JSON API for the Orvell Pulse platform: auth, stock/inventory, item

# orvell-pulse-backend — Laravel 12 REST API

## Purpose
Stateless JSON API for the Orvell Pulse platform: auth, stock/inventory, item
categories, customers, orders, Stripe payments, and WhatsApp "agent" provisioning.
Token auth via Laravel Sanctum; consumed by the Next.js frontend in `../orvell-pulse`.

## Key files
- `routes/api.php` — all API endpoints (`/api` prefix); authoritative route map.
- `bootstrap/app.php` — Laravel 12 wiring: routing, global `HandleCors`, exceptions, `/up` health.
- `app/Http/Controllers/` — thin request handlers (live auth in `API/AuthController.php`).
- `app/Services/` — business logic + `DB::transaction`.
- `app/Models/` — Eloquent models.
- `app/Http/Request/`, `app/Http/Resources/` — validation + JSON output shaping.
- `app/Traits/ResponseTrait.php` — `success()`/`fail()` response envelope.
- `config/`, `.env` — framework + integration config (sqlite default).
- `database/migrations/` — schema (⚠ incomplete; see `../docs/CODEBASE_AUDIT.md`).

## Data flow
HTTP → `public/index.php` → `bootstrap/app.php` → `routes/api.php` (+`auth:sanctum`)
→ Controller → FormRequest validate → Service (`DB::transaction`) → Eloquent Model
→ Resource / ResponseTrait → JSON response.

## Dependencies
External: MySQL (prod) / sqlite (default), Redis (OTP cache), Stripe, WHAPI.cloud,
SchoolEXL OTP API. Depended on by the Next.js frontend (`../orvell-pulse`).

## Conventions
PHP 8.2, PSR-4, thin controllers + service layer, bearer tokens (no sessions/CSRF).
Follow `../docs/PATTERNS.md`. ⚠ Known drift: ~10 referenced tables lack migrations;
secrets read via `env()` instead of `config()`.

## Common commands
- `composer install` · `php artisan key:generate` · `php artisan migrate`
- `php artisan serve` (or `composer dev` = serve + queue:listen + pail + vite)
- `composer test` / `php artisan test`
- `php artisan route:list` · `php artisan l5-swagger:generate`

# Important Rules

## Frontend Design Rules (Impeccable)
- For ANY frontend/UI work, run `/impeccable audit` after `/review` and `/impeccable polish` before final commit
- Never use Inter, Arial, Roboto, or system fonts as primary typeface — pick distinctive fonts
- Never use pure gray — always tint neutrals toward the brand color
- Never nest cards inside cards
- Never use gray text on colored backgrounds — check contrast
- Never use purple gradients as default — commit to a project-specific color palette
- Never use bounce/elastic easing — it feels dated
- See .claude/skills/impeccable/ for full design reference

---
> Source: [Sandeeptechnobren/orvell-pulse-backend](https://github.com/Sandeeptechnobren/orvell-pulse-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
