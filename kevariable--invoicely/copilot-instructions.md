## invoicely

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- **Laravel 12** on PHP 8.2+ (Dockerfile uses 8.4, CI tests on 8.4) with **Octane + FrankenPHP** in production.
- **Filament 3** is the primary admin UI, mounted at `/admin`. The root route (`/`) just redirects there.
- **Livewire/Volt + Flux** ship from the Livewire starter kit; auth scaffolding is present but `routes/auth.php` is currently *not* required from `routes/web.php`. Auth flows go through Filament's `->login()`.
- **Tailwind v4 + Vite 7** for assets.
- **DomPDF** (`barryvdh/laravel-dompdf`) is the active PDF renderer. `spatie/browsershot` + Puppeteer are installed and the Browsershot path is left commented in `GenerateInvoiceAction` — switching back means re-enabling Chromium (`docker/Dockerfile` already installs `chromium-headless-shell`).
- SQLite by default (`DB_CONNECTION=sqlite`); migrations also work against Postgres (Dockerfile installs `pdo_pgsql`).

## Common commands

```bash
# All-in-one dev (server + queue + pail logs + vite, via concurrently)
composer dev

# Individual processes
php artisan serve
npm run dev
php artisan queue:listen --tries=1
php artisan pail

# Tests (Pest). composer test clears config first.
composer test
./vendor/bin/pest                       # direct (matches CI)
./vendor/bin/pest tests/Feature/Auth    # single dir / file
./vendor/bin/pest --filter=DashboardTest
php artisan test --filter='it does X'   # by test name

# Lint / format (Pint runs in CI; no auto-commit)
vendor/bin/pint
vendor/bin/pint --test                  # check only

# Assets
npm run build

# Docker (FrankenPHP + Octane on host port 8989)
docker compose up --build
```

`phpunit.xml` forces `DB_CONNECTION=sqlite` with `:memory:` for tests, plus `array` cache/mail and `sync` queue — no real DB or services needed to run the suite.

CI requires Flux Pro credentials (`FLUX_USERNAME` / `FLUX_LICENSE_KEY` env/secrets) to install `livewire/flux` from `composer.fluxui.dev`; local `composer install` will need the same auth configured.

## Architecture

### Two parallel namespaces: `App\` and `Invoice\`

`composer.json` autoloads two PSR-4 roots:

- `App\` → `app/` — standard Laravel: Eloquent models, Filament resources, controllers, Livewire actions, mail. **This is where state lives and where the app actually executes.**
- `Invoice\` → `src/` — a domain layer (DDD-flavoured, organised as bounded contexts `Customer/` and `Invoice/`, each split into `Application/Data` DTOs and `Domain/{Actions,Contracts,Primitives}`).

The two layers do **not** form a hexagonal architecture in practice yet:

- `Domain/Contracts/*` interfaces (`CreateInvoice`, `GetInvoice`, `UpdateInvoice`, `GetCustomer`) are declared but **have no concrete bindings** — Filament resources and the `InvoicePreviewController` talk to Eloquent models directly.
- The one piece of `src/` that is wired in is `Invoice\Invoice\Domain\Actions\GenerateInvoiceAction`, instantiated with `new` from `InvoicePreviewController` and `InvoiceResource` to render the PDF.
- DTOs in `src/.../Application/Data/*` extend `Invoice\Base\DataReadonly` and hydrate from arrays via Valinor (`DataHydration::fromArray`); value objects extend `Invoice\Base\ValueObject` (`make()` / `toValue()` / `equals()`).

When adding new behaviour, follow the existing pattern: business state on the Eloquent model (`App\Models\Invoice` already owns invoice-number generation, amount calculation, public-token issuance, and view-state transitions), domain helpers in `src/` only when they would be reused outside Filament. Don't introduce a service-binding layer just to satisfy the contract interfaces unless you're also wiring concrete implementations.

### Invoice flow

1. **Authoring** happens entirely inside Filament: `app/Filament/Resources/InvoiceResource.php` + `RelationManagers/ItemsRelationManager.php` for line items.
2. `Invoice::generateInvoiceNumber()` builds the next number using `CompanySetting::getSettings()->invoice_prefix` + zero-padded id-based counter. **Note the parsing assumes a 4-character prefix** (`substr($lastInvoice->invoice_number, 4)`) — changing prefix length breaks numbering.
3. `Invoice` model boot hooks recompute `subtotal` / `total_amount` from `items` on every `creating`/`updating`. After mutating items, call `$invoice->updateAmounts()` to persist.
4. **Public sharing** — `Invoice::generatePublicToken()` issues a 64-char hex token; routes `/invoice/preview/{token}` and `/invoice/preview/{token}/download` (in `routes/web.php`, no auth) render the public view and PDF via `InvoicePreviewController`. First view flips `view_state` from `unread` → `viewed`.
5. **PDF** — `GenerateInvoiceAction::execute()` renders `resources/views/invoice-pdf-dompdf.blade.php` and pipes it through DomPDF. There are three invoice blade templates (`invoice-pdf.blade.php`, `invoice-pdf-dompdf.blade.php`, `invoice-pdf-tailwind.blade.php`); only the `-dompdf` one is currently rendered for downloads.
6. **Email notifications** — `App\Mail\InvoiceNotification` is the mailable used to send the share link.

### Settings singleton

`CompanySetting::getSettings()` is `firstOrCreate([])` — a true single-row singleton. Anywhere you need company info (PDF, invoice numbering, currency defaults), pull it through this method rather than querying `CompanySetting` directly.

### Currency

`App\Helpers\CurrencyHelper` is the source of truth: a static `CURRENCIES` array (USD, GBP at the time of writing) + symbol/format helpers. Filament forms feed it via `getSelectOptions()` / `getDefaultCurrency()`. Add new currencies there, not in migrations.

### Filament panel

`app/Providers/Filament/AdminPanelProvider.php` configures the only panel: id `admin`, top navigation, `Color::Neutral` palette, brand logos at `public/logo.png` / `public/dark_logo.png`. Resources/pages/widgets are auto-discovered from `app/Filament/{Resources,Pages,Widgets}`.

## Conventions worth preserving

- Match the existing split: persistent state and Filament-facing logic on Eloquent models in `app/Models/`; reusable cross-context primitives in `src/`. Don't move invoice calculations into the domain layer without also moving callers.
- New value objects: extend `Invoice\Base\ValueObject` and implement `make()` + `toValue()`. New DTOs: extend `Invoice\Base\DataReadonly` (camelCase props auto-convert to snake_case in `toArray()`).
- Pint is enforced in CI but the auto-commit step is disabled — run `vendor/bin/pint` locally before pushing.
- Tests live under `tests/Feature` (extends `Tests\TestCase`) and `tests/Unit`. `RefreshDatabase` is **not** applied globally in `tests/Pest.php` — opt in per file when needed.

---
> Source: [kevariable/invoicely](https://github.com/kevariable/invoicely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
