## php-laravel

> Laravel backend — routes, Eloquent, tests, security


# Laravel Backend

## Architecture
- Web routes + controllers for app features; session authentication.
- Authorization: policies + middleware. Validation: Form Requests.
- Business logic in services; keep controllers thin. Efficient Eloquent; avoid N+1.
- Migrations for all schema changes. Tables/columns: English `snake_case`.

## Tests
| Dir | Use |
|-----|-----|
| `tests/Feature/` | Routes, controllers, user flows |
| `tests/Unit/` | Models, services, helpers |

Class names: `PascalCase` + `Test` suffix. Run: `make test`.

## Security & GDPR
- Rate limit sensitive routes (`throttle` / project middleware).
- Log IP as SHA256 + salt `ADV_THROTTLE_SALT` only — no plaintext PII in logs.
- Privacy policy changes: see rule `gdpr-privacy` when editing legal views/config.

## Before new code
Search models, services, controllers. Example: balance logic → `Household::isDebtBalancingMode()`, `isSharedWalletMode()`.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
