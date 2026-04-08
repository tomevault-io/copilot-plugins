## pharmacy-pos-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup
composer setup

# Run development servers (Laravel + queue + logs + Vite concurrently)
composer dev

# Run all tests
composer test

# Run a single test file
php artisan test tests/Feature/GenericTest.php

# Run a specific test by name (Pest)
php artisan test --filter "shows a list of generics"

# Code formatting (Laravel Pint)
./vendor/bin/pint

# Database migrations
php artisan migrate

# Tail logs
php artisan pail
```

## Docker / Sail

The project uses Laravel Sail for Docker-based development:

```bash
# Start containers
./vendor/bin/sail up -d

# Run artisan inside container
./vendor/bin/sail artisan migrate

# Run tests inside container
./vendor/bin/sail test
```

Docker Compose (`compose.yaml`) spins up:
- **laravel.test** — PHP 8.5 app container (ports 80, 5173)
- **mysql** — MySQL 8.4 (forwarded to host port 3307)

The test environment (`phpunit.xml`) uses `DB_DATABASE=testing` and `RefreshDatabase` — tests hit a real database, not mocks.

## Architecture

This is a **Laravel 12 REST API backend** for a pharmacy point-of-sale system. It uses Sanctum for auth (token-based) and Pest for testing.

### Request lifecycle

`routes/api.php` → `Controller` → `FormRequest` (validation) → `Model` (Eloquent) → JSON response

Each domain resource follows this exact pattern with no deviation:

| Layer | Location | Purpose |
|---|---|---|
| Route | `routes/api.php` | `Route::apiResource(...)` |
| Controller | `app/Http/Controllers/` | CRUD via Eloquent, returns JSON |
| Form Requests | `app/Http/Requests/` | Authorization + validation rules |
| Model | `app/Models/` | Eloquent model with `$fillable` |
| Policy | `app/Policies/` | Authorization gates (currently all open) |
| Migration | `database/migrations/` | Schema definition |
| Factory | `database/factories/` | Test data generation |
| Tests | `tests/Feature/` | Feature tests per resource |

### Domain models

- **Generic** — drug generic names (unique `name`, optional `description`) — fully implemented with controller, requests, policy, factory, and feature tests
- **Product** — pharmacy product (schema TBD, migration is a stub)
- **Inventory** — stock tracking (schema TBD, migration is a stub)
- **Supplier** — product suppliers
- **Manufacturer** — drug manufacturers
- **Pharmacy** — pharmacy locations
- **Document** — documents/records
- **User** — auth via Laravel Breeze + Sanctum

### Adding a new resource

Follow the `Generic` resource as the canonical pattern:
1. Controller returning paginated JSON (`paginate(10)`) on index
2. `StoreXxxRequest` and `UpdateXxxRequest` with `authorize()` and `rules()`
3. Model with `$fillable`
4. Policy class
5. `Route::apiResource(...)` entry in `routes/api.php`
6. Feature test covering index, store (valid + duplicate/invalid), update, delete

### Testing conventions

- All feature tests use `RefreshDatabase` (configured globally in `tests/Pest.php`)
- Tests use `->getJson()`, `->postJson()`, etc. with named routes (`route('generics.index')`)
- Factories are required for every model used in tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yurimarine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yurimarine)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
