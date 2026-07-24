## screeenly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**PHP / Backend**
```bash
composer format                 # Auto-format PHP code (PHP-CS-Fixer)
vendor/bin/phpunit              # Run full test suite
vendor/bin/phpunit --filter TestName   # Run a single test
php artisan migrate             # Run database migrations
php artisan storage:link        # Link public storage (auto-runs on install/update)
```

**Frontend**
```bash
npm run dev                     # Build assets (development)
npm run prod                    # Build assets (production)
npm run watch                   # Watch + rebuild on change
```

**CI runs** PHP 8.0 on Ubuntu with Puppeteer installed; mirror that locally for screenshot tests.

## Architecture

Screeenly is a Laravel 8 screenshot-as-a-service app. The core feature logic lives in `/modules/Screeenly/` rather than `/app/` — that namespace is the primary place to work.

### Request flow (API)

```
POST /api/v1/fullsize  or  /api/v2/screenshot
  → ScreenshotController  (modules/Screeenly/Http/Controllers/Api/)
  → CreateScreenshotRequest  (validation)
  → CaptureService  (modules/Screeenly/Services/)
  → ChromeBrowser  or  AwsBrowser  (selected via DI binding in ScreeenlyServiceProvider)
  → Spatie Browsershot + Puppeteer
  → saved to SCREEENLY_DISK (local public or S3)
  → JSON response: screenshot URL + metadata
```

### Key directories

| Path | Purpose |
|---|---|
| `modules/Screeenly/Services/` | Core business logic — `CaptureService`, `ChromeBrowser`, `AwsBrowser`, `InMemoryBrowser` |
| `modules/Screeenly/Contracts/` | `CanCaptureScreenshot` interface — the abstraction point for swapping browser backends |
| `modules/Screeenly/Entities/` | Value objects: `Url`, `Screenshot` |
| `modules/Screeenly/Http/Controllers/Api/` | API v1 and v2 controllers |
| `modules/Screeenly/Http/Controllers/App/` | Web UI: dashboard, settings, OAuth |
| `modules/Screeenly/Guards/` | `ScreeenlyTokenGuard` — custom auth guard for API key authentication |
| `modules/Screeenly/Providers/ScreeenlyServiceProvider.php` | Binds `CanCaptureScreenshot` to the correct browser implementation |
| `config/screeenly.php` | App-specific config: sandbox, disk, Lambda mode |
| `tests/Modules/Screeenly/` | PHPUnit feature tests |

### Dual browser backends

- **Local** (`ChromeBrowser`): uses `spatie/browsershot` + local Puppeteer/Node.
- **Lambda** (`AwsBrowser`): uses `hammerstone/sidecar` to invoke AWS Lambda — enabled via `SCREEENLY_USE_AWS_LAMBDA_BROWSER=true`.
- **Test** (`InMemoryBrowser`): used in tests; bound automatically when `APP_ENV=testing`.

The binding is resolved in `ScreeenlyServiceProvider`. To add a new backend, implement `CanCaptureScreenshot` and register it there.

### API versioning

v1 and v2 API controllers coexist under `modules/Screeenly/Http/Controllers/Api/`. v2 is the current preferred API.

### Authentication

Two guards:
- `web` — standard Laravel session auth; used for the dashboard. GitHub OAuth via `laravel/socialite`.
- `screeenly-token` — custom guard in `modules/Screeenly/Guards/`; validates `Authorization: Bearer <api-key>` headers against the `ApiKey` model.

### Code style

PHP-CS-Fixer config at `.php_cs`: PSR-2, short arrays, ordered imports, return type declarations. Run `composer format` before committing.

---
> Source: [stefanzweifel/screeenly](https://github.com/stefanzweifel/screeenly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
