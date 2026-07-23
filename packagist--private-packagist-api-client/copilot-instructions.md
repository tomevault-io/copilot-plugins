## private-packagist-api-client

> PHP API client for Private Packagist.

# Private Packagist API Client

PHP API client for Private Packagist.

## Commands

- `composer test` — Run tests
- `composer cs-fix` — Fix code style (src + tests)
- `composer analyze` — Run PHPStan static analysis

## Architecture

- `src/Api/AbstractApi.php` — Base class for all API endpoints. Provides `get()` for single resources and `getCollection()` for list endpoints with auto-pagination.
- `src/Api/*.php` — API endpoint classes (Packages, Teams, Customers, etc.)
- `src/Api/*/` — Nested API endpoints (e.g. `Customers/VendorBundles`, `Packages/Artifacts`)
- `src/Client.php` — Entry point, provides access to all API classes

## Notes

- Supports PHP 7.2+ — avoid modern syntax not available in 7.2

## Conventions

- List endpoints use `getCollection()` (handles pagination and default limit)
- Single-resource endpoints use `get()`

---
> Source: [packagist/private-packagist-api-client](https://github.com/packagist/private-packagist-api-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
