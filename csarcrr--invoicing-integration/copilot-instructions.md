## invoicing-integration

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
composer test        # Run Pest tests in parallel
composer analyse     # Run PHPStan static analysis (level 7)
composer format      # Run Laravel Pint code formatter
composer complete    # Run all: format + analyse + test
```

To run a single test file:
```bash
./vendor/bin/pest tests/Unit/Invoice/CreateTest.php
```

## Architecture

This is a **Laravel package** (`csarcrr/invoicing-integration`) providing a provider-agnostic API for Portuguese fiscal invoicing, currently supporting Cegid Vendus.

### Request Flow

User calls a Facade → Action class → Provider implementation → HTTP → Provider API

```
Facades/Invoice::create(InvoiceData)
  → Actions/InvoiceAction (match on Provider enum)
    → Provider/CegidVendus/Invoice/Create
      → buildPayload() assembles request
        → Http::provider() macro (configured HTTP client)
```

### Key Directories

- **`src/Actions/`** — Orchestrators that route operations to the correct provider implementation via `match` on `Provider` enum
- **`src/Contracts/`** — Interfaces that all provider implementations must satisfy (e.g., `ShouldCreateInvoice`, `CreateClient`)
- **`src/Data/`** — Spatie Laravel Data DTOs (`InvoiceData`, `ItemData`, `ClientData`, etc.) — validated on instantiation
- **`src/Enums/`** — Strongly-typed domain enums (`InvoiceType`, `ItemTax`, `TaxExemptionReason`, `PaymentMethod`, `Provider`)
- **`src/Provider/CegidVendus/`** — All Cegid Vendus-specific logic, mirroring the `Invoice/`, `Client/`, `Item/` sub-structure
- **`src/Exceptions/`** — Domain exceptions organized by concern (`Providers/`, `Invoice/`, `Pagination/`)
- **`src/Traits/`** — `HasMakeValidation`, `HasPaginator`, `HasConfig`, `EnumOptions`

### Adding a New Provider

1. Add a case to `Enums/Provider`
2. Add provider config to `config/invoicing-integration.php`
3. Implement the relevant contracts (e.g., `ShouldCreateInvoice`) in `src/Provider/<ProviderName>/`
4. Add the provider case to the `match` in each `Action` class

### HTTP Layer

Two custom HTTP macros registered in the service provider:
- `Http::provider()` — Returns a pre-configured HTTP client for the active provider
- `Http::handleUnwantedFailures()` — Centralized mapping of HTTP status codes to domain exceptions

### DTOs

All domain data uses `Spatie\LaravelData`. DTOs validate on construction. Use `Optional` (from `src/Helpers/Properties.php`) when a field may be intentionally absent vs. `null`.

### Code Requirements

- PHP 8.2+, always include `declare(strict_types=1);` at the top of every PHP file
- PHPStan level 7 — run `composer analyse` before committing
- Supports Laravel 11.x and 12.x; tests run against both via Orchestra Testbench

### Namespaces

| Path | Namespace |
|---|---|
| `src/` | `CsarCrr\InvoicingIntegration\` |
| `tests/` | `CsarCrr\InvoicingIntegration\Tests\` |
| `database/factories/` | `CsarCrr\InvoicingIntegration\Database\Factories\` |
| `workbench/app/` | `Workbench\App\` |

### Testing Conventions

- Write tests using Pest syntax — `it()`, `test()`, `expect()` — never raw PHPUnit
- Use `pestphp/pest-plugin-arch` for architecture tests
- Tests mirror `src/` structure under `tests/Unit/`

### Service Provider

`CsarCrr\InvoicingIntegration\InvoicingIntegrationServiceProvider`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/csarcrr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/csarcrr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
