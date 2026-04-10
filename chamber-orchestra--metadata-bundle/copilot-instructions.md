## metadata-bundle

> This is a PHP 8.5 library with PSR-4 autoloading (`Dev\\MetadataBundle\\` mapped to the repo root).

# Repository Guidelines

## Project Structure & Module Organization
This is a PHP 8.5 library with PSR-4 autoloading (`Dev\\MetadataBundle\\` mapped to the repo root).
Key directories:
- `src/` holds the library code, organized by feature areas such as `DependencyInjection/`, `EventSubscriber/`, `Mapping/`, and `Reader/`.
- `src/Resources/` contains framework resources and configuration assets.
- `tests/Integrational/` and `tests/Unit/` hold PHPUnit tests (current coverage is minimal).
- `bin/phpunit` is the local PHPUnit runner.

## Build, Test, and Development Commands
Use Composer to install dependencies and run tests:
- `composer install` installs PHP dependencies into `vendor/`.
- `bin/phpunit` runs the test suite using `phpunit.xml.dist` (bootstraps `vendor/autoload.php`).
- `bin/phpunit --filter SomeTest` runs a focused subset of tests by class or method.

## Coding Style & Naming Conventions
- Indentation: 4 spaces, no tabs.
- Follow PSR-12 conventions for PHP formatting.
- Names: `PascalCase` for classes (`MetadataReader`), `camelCase` for methods (`loadMetadata`), and `snake_case` only when required by external APIs.
- Keep namespaces aligned with directory structure under `src/`.

## Testing Guidelines
- Framework: PHPUnit (configured in `phpunit.xml.dist`).
- Test locations: place unit tests in `tests/Unit/` and integration tests in `tests/Integrational/`.
- Naming: suffix test classes with `Test` (e.g., `ReaderTest`).
- Run all tests with `bin/phpunit` before opening a PR.

## Commit & Pull Request Guidelines
- There is no established commit message convention yet; use clear, imperative messages (e.g., "Add metadata reader support").
- PRs should include a short description, the rationale for changes, and any relevant test evidence (command output or notes).

## Configuration Tips
- PHPUnit sets `APP_ENV=test` and `APP_DEBUG=true` for tests; avoid hard-coding environment values in production code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chamber-orchestra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chamber-orchestra)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
