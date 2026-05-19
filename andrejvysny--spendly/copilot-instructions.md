## php-laravel

> PHP/Laravel conventions for app, database, routes, config.


# PHP/Laravel

- Use `declare(strict_types=1);` in all PHP files.
- Follow PSR-12. Use Form Requests for validation; keep controllers thin and delegate to services.
- Inertia: `Inertia::render('page/name', [...])` — page name must match path under `resources/js/pages` (e.g. `accounts/index`, `dashboard`).
- Use Laravel Policies for authorization (`$this->authorize()`).
- Reference: app/Http/Controllers/Controller.php, app/Http/Controllers/Transactions/TransactionController.php (index + Inertia pattern).

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
