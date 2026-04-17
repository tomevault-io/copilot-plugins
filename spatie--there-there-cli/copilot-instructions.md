## there-there-cli

> A standalone CLI tool for [There There](https://there-there.app) built on Laravel Zero. Uses `spatie/laravel-openapi-cli` to auto-generate artisan commands from the There There OpenAPI spec.

# There There CLI

## Project overview

A standalone CLI tool for [There There](https://there-there.app) built on Laravel Zero. Uses `spatie/laravel-openapi-cli` to auto-generate artisan commands from the There There OpenAPI spec.

- **Package**: `spatie/there-there-cli`
- **Binary**: `there-there`
- **Install**: `composer global require spatie/there-there-cli`

## Architecture

- **Laravel Zero 12** — PHP CLI micro-framework
- **spatie/laravel-openapi-cli** — reads `resources/openapi.yaml` and registers one command per API endpoint using `operationId`-based naming
- **CredentialStore** (`app/Services/CredentialStore.php`) — reads/writes API tokens and profiles to `~/.there-there/config.json`
- **LoginCommand / LogoutCommand** — custom commands (not from OpenAPI spec) for auth flow
- **Profiles** — multiple workspaces supported via named profiles. Config stores profiles under a `profiles` key with a `default_profile` pointer. All commands accept `--profile` (resolved from argv in CredentialStore constructor).

## Key files

- `there-there` — CLI entry point (the binary)
- `app/Providers/AppServiceProvider.php` — registers CredentialStore singleton and OpenApiCli
- `app/Services/CredentialStore.php` — credential persistence to `~/.there-there/config.json`
- `app/Commands/LoginCommand.php` — `there-there login`
- `app/Commands/LogoutCommand.php` — `there-there logout`
- `resources/openapi.yaml` — bundled There There API spec
- `config/app.php` — providers list (must manually register `OpenApiCliServiceProvider`)
- `box.json` — PHAR build config (must include `resources` directory)

## AI agent skill

When updating the AI agent skill (e.g. filters, sorts, available commands), always verify against the actual `there-there` build (`php there-there list`, `php there-there <command> --help`) to avoid documenting incorrect flags, filter names, or sort options.

## Important notes

- Laravel Zero disables package auto-discovery. Any package service providers must be registered manually in `config/app.php`.
- The `.auth()` callable (not `.bearer()`) is used for dynamic credential resolution from `CredentialStore`.
- The `resources/` directory must be in `box.json` `directories` for the spec to be bundled in the PHAR.
- `resource_path()` resolves to `phar://` paths when running inside a PHAR — this works for reading but not writing.

## Coding standards

Follow PSR-12. Use Laravel/Pint for formatting. Tests use Pest.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spatie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
