## laravel-modules

> `sinemacula/laravel-modules` — A lightweight, convention-driven modular architecture package for Laravel. Auto-discovers

# Project Overview

`sinemacula/laravel-modules` — A lightweight, convention-driven modular architecture package for Laravel. Auto-discovers
modules as directories with zero manifests, zero boilerplate, and nothing new to learn.

This package extracts the module discovery and provider infrastructure from the `sinemacula/laravel-modular-template`
into a reusable Composer package. The template then becomes a thin boilerplate that depends on this package.

- **Namespace:** `SineMacula\Laravel\Modules`
- **Source:** `src/`
- **Type:** Library (Composer package)

## Commands

```bash
composer install              # Install dependencies
composer check                # Run qlty static analysis (PHPStan level 8, PHP-CS-Fixer, CodeSniffer, etc.)
composer check -- --all --no-cache --fix  # Checks with auto-fix
composer format               # Format code via qlty
composer test                 # Run tests (Paratest, parallel execution)
composer test-coverage        # Run tests with clover coverage report

# Single test file
vendor/bin/phpunit tests/Unit/SomeTest.php

# Single test method
vendor/bin/phpunit --filter testMethodName tests/Unit/SomeTest.php
```

## Conventions

- Default branch: `master`. Branch prefixes: `feature/`, `bugfix/`, `hotfix/`, `refactor/`, `chore/`
- Use Conventional Commits
- Never mention AI tools in commit messages or code comments
- PHPStan level 8 (strict). All code must pass `composer check` before handoff
- Run `composer test` before handoff when executable PHP changes are made
- Keep changes minimal and scoped to the request; avoid unrelated refactors
- Do not change static analysis or formatting configuration without approval

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sinemacula) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
