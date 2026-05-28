## laravel-php-monolithic-boilerplate

> This repository is a Laravel modular monolith boilerplate.

# Codex instructions

This repository is a Laravel modular monolith boilerplate.

## Commands

```bash
composer install
cp .env.example .env
php artisan key:generate
php artisan test
vendor/bin/pint
php artisan serve --host=0.0.0.0 --port=8000
```

## Project conventions

- Keep the app monolithic: one deployable Laravel application.
- Organize new features under `app/Modules/{Feature}`.
- Follow `Controller → Request → Service → Repository`.
- Keep controllers thin.
- Keep business logic in services.
- Keep persistence behind repository interfaces.
- Return API responses with `App\Common\Api\ApiResponse`.
- Throw `App\Common\Exceptions\AppException` for expected domain errors.
- Use Form Request classes for validation.
- Add feature tests for every new endpoint.

## Do not

- Do not commit `.env`.
- Do not put secrets in config files.
- Do not put business rules in controllers.
- Do not couple feature modules through direct cross-module repository access unless unavoidable.

---
> Source: [APPNEURAL-Boilerplates/laravel-php-monolithic-boilerplate](https://github.com/APPNEURAL-Boilerplates/laravel-php-monolithic-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
