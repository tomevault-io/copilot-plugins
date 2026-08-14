## zimatec-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ZiMaTech** is a Laravel 12 manufacturing/production management system for engineering, metalworking, and fabrication shops, localized in **German (`de`)**, with **`en`** as secondary locale. It is a **hybrid application** that runs as a standard web app, installs as a PWA on tablets/mobiles, or runs natively as an **Electron** desktop shell for shop-floor terminals.

It tracks employee time on machines/projects, manages customer projects with positions and services, tracks physical components (Bauteile) with measurements, handles suppliers, and generates PDF project offers with Microsoft Graph email integration. It also includes a weekly production scheduler, an internal AI project-assistant endpoint, a hardware helpdesk (printer problems), an internal feedback loop, and a multi-warehouse system (Lager).

## Common Commands

```bash
# Initial setup (one-shot: install deps, .env, key, migrate, npm, build)
composer run setup

# Per-project link for uploaded PDFs / component images
php artisan storage:link

# Development (server + queue listener + vite concurrently, color-tagged logs)
composer run dev
#   → http://127.0.0.1:8000

# Electron desktop shell
npm run start             # launches electron/main.js (auto-starts php artisan serve if not running)
npm run build-electron    # packages Windows NSIS installer into dist/

# Testing (Pest)
composer run test         # clears config cache then runs Pest
php artisan test          # direct
./vendor/bin/pest tests/Feature/MachineLogsTest.php   # single test file
./vendor/bin/pest --filter=MethodName                 # single test method

# Code quality
./vendor/bin/pint         # Laravel Pint formatter
```

Tests require a MySQL test database named `zimatech_test` (configured in `phpunit.xml`). In-memory drivers are used for cache/session/queue; mail is captured to an array.

## Architecture

### Module Flags (`config/modules.php`)

Toggles for routes/controllers — checked with `if (config('modules.foo'))` before wiring endpoints:

| Flag | Scope | Default |
| :--- | :--- | :--- |
| `teams` | User & role management | `true` |
| `projects` | Projects, Bauteile, positions, services | `true` |
| `time` | Time tracking on machines | `true` |
| `suppliers` | Supplier CRUD + offers + assigned projects | `true` |
| `settings` | Admin machine/project settings | `true` |
| `feedback` | Internal feedback (public ask endpoint) | `true` |
| `tablar` | Warehouse/Lager (multi-warehouse stock) | `true` |
| `scheduler` | Weekly production scheduler (`/scheduler`) | `true` |
| `project_offers` | Offer builder + PDF/email | `false` |
| `emails` | Microsoft Graph + IMAP integration | `false` |

### Controller Layout
- `app/Http/Controllers/` — public/user-facing (e.g. `TimeRecordController`, `ProjectController`, `SchedulerController`, `PrinterProblemController`, `TablarController`).
- `app/Http/Controllers/Admin/` — gated by `auth` + `role:admin` (middleware alias `role`, defined by `App\Http\Middleware\RoleMiddleware`); includes `AdminLagerController`, `ActivityTimelineController`, `FeedbackController`, etc.
- `app/Http/Controllers/Admin/Settings/` — admin settings sub-namespace (machines, machine status, project services/status, email templates, feature flags).

### Key Models (`app/Models/`)
- **Time domain:** `TimeRecord`, `TimeLog`, `TimeChangeRequest`, `Process`, `ProcessPause`, `Machine`, `MachineStatus`.
- **Project domain:** `Project`, `Position`, `ProjectService`, `ProjectStatus`, `Bauteil`, `BauteilMeasurement`, `Material`, `MaterialConsumption`.
- **Supplier domain:** `Supplier`, `SupplierOffer`, `SupplierProject`, `SupplierService`.
- **Offer domain:** `ProjectOffer`, `OfferCalculation`, `OfferCalculationItem`, `OfferFile`, `OfferEmail`, `EmailTemplate`.
- **Workflow:** `ProductionSchedule`, `Notification`, `Feedback`, `PrinterProblem`, `PrinterProblemAttachment`, `PrinterProblemEmail`, `Lager`.

### Routing
- All HTTP routes live in `routes/web.php` (Laravel 12 style — no `RouteServiceProvider` `mapWebRoutes`).
- Public routes (home, `/projects/index`, `/projects/logs`, `/parse-log`, `/feedback/*`, `/scheduler/*`) sit at the top.
- Authenticated routes (`printer-problems` resource + nested `attachments`/`emails`) and admin routes are grouped below.
- AI endpoint: `POST /assistant/ask-recommendations` (named `assistant.recommendations`) — handled by `HomeController::askRecommendations`.
- Locale switcher: `GET /language/{locale}` (supported: `de`, `en`) via `LanguageController`. The `SetLocale` middleware applies the saved/active locale.

### Database
- MySQL (config in `.env`). Migrations under `database/migrations/` (all dated `YYYY_MM_DD_HHMMSS_*.php`).
- Seeders: `DatabaseSeeder`, `MaterialLagerSeeder`.

## Key Integration Points

### Microsoft Graph (Outlook) — outbound
- `microsoft/microsoft-graph` v2 + `microsoft/kiota-authentication-phpleague` + `microsoft/kiota-http-guzzle` send project offer emails.
- Config in `config/services.php` (Azure AD credentials / tenant).

### IMAP — inbound
- `webklex/laravel-imap` v6 for receiving mail.
- Config in `config/imap.php`.

### PDF / Excel / PowerPoint
- `barryvdh/laravel-dompdf` for PDF (project offers, etc.).
- `phpoffice/phpspreadsheet` for Excel exports.
- `phpoffice/phppresentation` for PowerPoint (vendor use only).
- `intervention/image` 3.0 for image manipulation (component photos, etc.).

### Frontend Stack
- **Vite 7** + `laravel-vite-plugin` (entry: `resources/sass/app.scss` and JS imports).
- **Tailwind CSS v4** (`@tailwindcss/vite`) **+ Bootstrap 5** hybrid styling; Bootstrap Icons.
- **SweetAlert2** for confirmations/notifications.
- **PWA:** `public/manifest.json`, `public/sw.js`, `public/offline.html` for shop-floor offline shell.

### Desktop Shell
- `electron/main.js` checks `http://127.0.0.1:8000`; if unreachable, it spawns `php artisan serve` automatically and kills the child process on quit.
- The window loads `/projects` by default.
- Build target is NSIS Windows installer (`build.win.target` in `package.json`); packaged files are listed under `build.files`, with `storage/` mirrored to `extraResources`.

## Localization
- Primary: **German** (`lang/de/`). All UI strings, labels, and validation messages are German-first.
- Secondary: `lang/en/`, plus `lang/en.json` for any short keys.
- Switch via `GET /language/{locale}`; `SetLocale` middleware persists selection.

## Git Workflow
- `main` — production.
- `staging` — staging.
- Feature branches off either as needed.

---
> Source: [Hamzii982/zimatec_backend](https://github.com/Hamzii982/zimatec_backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
