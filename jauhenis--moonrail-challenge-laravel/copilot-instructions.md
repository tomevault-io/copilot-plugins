## moonrail-challenge-laravel

> This is a Laravel-based PHP application running on PHP 8.1 FPM, containerized with Docker. It uses a Service-Controller-Model architecture.

# Moonrail Project Rules

## Project Overview
This is a Laravel-based PHP application running on PHP 8.1 FPM, containerized with Docker. It uses a Service-Controller-Model architecture.

## Architecture & Coding Standards
- **Controllers**: Keep controllers thin. Use them for request handling and response formatting. Inject services for business logic.
- **Services**: All business logic must reside in Service classes (e.g., `App\Services\PlayerService`).
- **Models**: Use Eloquent models with proper type hinting and casts.
- **Resources**: Use API Resources (`App\Http\Resources`) for transforming models to JSON responses.
- **Requests**: Use Form Requests (`App\Http\Requests`) for validation logic.
- **Enums**: Use PHP 8.1 Enums for fixed sets of values (e.g., `App\Enums\PlayerPosition`).
- **Codestyle**: Follow PSR-12. Use 4 spaces for indentation.

## PHP 8.1 Standards
- **Do not add comments**: Never add your own comments.
- **Type Hinting**: Use scalar type hints for all function arguments and return types. Use nullable types (`?string`) or union types (`string|int`) where appropriate.
- **Enums**: Use Backed Enums (usually `string`) for all fixed sets of values. Access values via `->value`.
- **Constructor Property Promotion**: Use property promotion in constructors for cleaner dependency injection (e.g., `public function __construct(protected PlayerService $playerService) {}`).
- **Readonly Properties**: Use `readonly` for properties that should not be modified after initialization.
- **Attributes**: Use PHP 8.1 attributes where applicable (e.g., for routing or validation if the project evolves that way).

## Docker & Environment
- **ALL** commands (artisan, composer, etc.) MUST be run via the Docker container.
- Use `docker compose exec app <command>` or the provided `Makefile` shortcuts.
- **Default Ports**:
  - App: `3000`
  - Redis: `6389`
- **PHP FPM**: The project runs on PHP 8.1 FPM.

## Makefile Instructions
Use the following `make` commands for common tasks:
- `make optimize`: Run `php artisan optimize`
- `make rebuild-restart`: Rebuild and restart containers
- `make composer-install`: Run `composer install`
- `make composer-dump-autoload`: Run `composer dump-autoload`
- `make env-setup`: Setup `.env` and generate app key

## API & Middleware
- **Base URL**: All API routes are prefixed with `/api`.
- **Authentication**:
  - Some routes use `auth:sanctum`.
  - Specific routes (like delete) use the custom `bearer_token` middleware.
- **Middleware**: Custom middleware is located in `app/Http/Middleware`. Register new middleware in `app/Http/Kernel.php`.

## Testing
- **Framework**: Use PHPUnit for testing.
- **Unit tests**: `tests/Unit`. Focus on testing individual classes (e.g., Services) in isolation.
- **Feature tests**: `tests/Feature`. Focus on testing API endpoints and end-to-end flows.
- **Execution**: Run tests via docker: `docker compose exec app php artisan test`.
- **Database**: Use the `RefreshDatabase` trait for tests that interact with the database.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JauheniS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
