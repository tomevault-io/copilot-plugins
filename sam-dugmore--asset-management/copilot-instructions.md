## asset-management

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tech asset management system built with Laravel 13. Tracks IT assets (laptops, equipment) assigned to people and locations.

## Commands

Tests run directly with local PHP (SQLite in-memory). Other commands use Sail:

```bash
# Start environment (for app, database, etc.)
./vendor/bin/sail up -d

# Run all tests (local PHP, no Sail needed)
php artisan test

# Run a single test file
php artisan test tests/Feature/Http/AssetControllerTest.php

# Run a specific test method
php artisan test --filter=testAssetIndex

# Migrations & seeding
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan db:seed

# Code style (Laravel Pint)
./vendor/bin/sail exec laravel.test ./vendor/bin/pint

# Mess detection (PHPMD)
./vendor/bin/phpmd app text phpmd.xml

# Static analysis (Larastan/PHPStan)
./vendor/bin/phpstan analyse

# Code sniffing (PHPCS — PSR-12)
./vendor/bin/phpcs              # check
./vendor/bin/phpcbf             # auto-fix

# Test coverage
php artisan test --coverage

# Frontend assets
npm run dev    # dev server (Vite on port 5173)
npm run build  # production build
```

## Architecture

### Models

- **Asset** — core entity; belongs to `Person` and `Location`; has `assigned()`/`unassigned()` query scopes; changes tracked via `AssetObserver` → `AssetAuditLog`
- **Person** — employees who own assets
- **Location** — physical locations; has `full_location` computed attribute
- **User** — system users; has static `systemUser()` method used for audit logging; roles/permissions via Spatie
- **AssetAuditLog** — immutable log of field-level changes to assets
- **CheckoutRequest** — self-service checkout/reassignment requests with status workflow; soft deletes
- **Setting** — key/value app settings; accessed via `Setting::get($key, $default)`

### Controllers (Web)

Resource controllers with no `show` route on any entity. Routes: index, create, store, edit, update, destroy.

- `AssetController` — index supports sorting, searching, pagination (20/page), statistics, and recent activity
- `PersonController`, `LocationController` — standard CRUD
- `ExportController` — triggers `ExportAssetsJob` (queued), serves download when ready
- `ImportController` — CSV import via `ImportAssetsJob`; requires `create` asset permission
- `CheckoutController` — self-service checkout flow (identity → asset selection → request submission)
- `CheckInController` — self-service asset check-in by identifier
- `CheckoutRequestController` — admin review/approve/deny checkout requests via `CheckoutApprovalService`
- `MyRequestsController` — public lookup of checkout requests by email
- `DashboardController` — renders dashboard (invokable)

### API (`app/Http/Controllers/Api/`)

REST API with Sanctum auth. Controllers use `ResolvesIncludes` trait for `?include=` query param.

- `AuthController` — login/logout/token management
- `AssetController`, `LocationController`, `PersonController` — CRUD resources with `AssetResource`/`LocationResource`/`PersonResource` API resources
- `CheckoutRequestController` — approve/deny with `CheckoutApprovalService`
- `ExportController`, `ImportController` — async export/import

### Services

- `CheckoutApprovalService` — handles checkout request approval: creates person/asset/location as needed, fires `CheckoutRequestApproved` event
- `CsvImportService` — parses and validates CSV import files

### Livewire Components

- `AssetTable`, `LocationTable`, `PeopleTable` — searchable/sortable data tables with pagination
- `DashboardStats` — summary counts for the dashboard
- `AssetsByLocationChart`, `RecentActivityChart` — dashboard charts
- `FloorPlan` — visual floor plan of assets by building/floor

### Filament Admin (`/admin`)

- `ManageSettings` — app settings page
- `UserResource` — user management
- `RoleResource` — role/permission management (Spatie)

### Events & Listeners

- `AssetReassigned` → `LogAssetReassignment`, `SendAssetReassignmentNotification`
- `CheckoutRequestApproved`

### Logging

- `dev` channel (default for local) — daily rotation (7 days), wrapped via `App\Logging\DeduplicateDailyLogs` tap so repeat warnings/errors collapse to one entry per request, with `App\Logging\RequestIdProcessor` injecting a per-request UUID into `extra.request_id`.

### Policies

Authorization via `AssetPolicy`, `CheckoutRequestPolicy`, `LocationPolicy`, `PersonPolicy`.

### Jobs & Notifications

- `ExportAssetsJob` → `AssetExportReadyNotification`
- `ImportAssetsJob` / `ImportAssetRowJob` → `AssetImportCompleteNotification`
- `AssetCheckedOutNotification`, `AssetMaintenanceNotification`
- Queue driver: `database`

### Form Requests

All in `app/Http/Requests/`: `StoreAssetRequest`, `UpdateAssetRequest`, `StoreLocationRequest`, `UpdateLocationRequest`, `StorePersonRequest`, `UpdatePersonRequest`, `StoreCheckoutRequestRequest`, `DenyCheckoutRequestRequest`.

### Frontend

Blade templates + Tailwind CSS v4 via Vite. Layout: `resources/views/layouts/app.blade.php`.

## Workflow

This is a **learning project** — do not post plans to GitHub issues and do not raise PRs. There are no issues to link to.

## Conventions

- **TDD**: write failing tests first, implement to pass
- No `show` route on any resource controller
- Eager load relationships with `with()` in controllers
- Tests use SQLite (in-memory) — see `phpunit.xml`
- API resources support sparse fieldsets via `FilterableFields` trait: `?fields[assets]=name,tag`
- Custom validation rules in `app/Rules/`: `AssetIdentifier` (asset tag/serial lookup), `FutureDate`

## Gotchas

- Laravel 13 uses `bootstrap/app.php` for routing/middleware/exceptions — there are no `Http/Kernel.php` or `Console/Kernel.php` files
- PHPMD, PHPStan, and PHPCS run outside Sail (local PHP); Pint runs inside Sail
- `AuthController` returns 418 if you send a bearer token to the login endpoint

---
> Source: [sam-dugmore/asset-management](https://github.com/sam-dugmore/asset-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
