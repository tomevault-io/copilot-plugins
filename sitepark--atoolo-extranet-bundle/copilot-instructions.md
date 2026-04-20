## atoolo-extranet-bundle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Symfony bundle that secures GraphQL and controller requests against unauthorized access for extranet sites. When `SITE_MODE=extranet` is set, the bundle enforces `ROLE_WEB_ACCOUNT` authentication on all controller and GraphQL requests.

## Commands

```bash
# Run all tests with coverage
composer test

# Run PHPUnit directly (single test class)
./tools/phpunit.phar -c phpunit.xml tests/path/to/SomeTest.php

# Run mutation testing
composer test:infection

# Run all static analysis tools
composer analyse

# Run individual analysis tools
composer analyse:phpstan       # PHPStan level 9
composer analyse:phpcsfixer    # Code style check (PER-CS standard)
composer analyse:phplint       # PHP syntax linting
composer analyse:compatibilitycheck  # PHP 8.1–8.4 compatibility

# Auto-fix code style
composer cs-fix
```

## Architecture

```
src/
├── AtooloExtranetBundle.php                    # Bundle entry point; loads config/services.yaml
└── Security/
    ├── ControllerDefaultSecurityListener.php   # KernelEvents::CONTROLLER subscriber
    └── GraphQlDefaultAccessConfigProcessor.php # OverblogGraphQL ConfigProcessorInterface
```

**ControllerDefaultSecurityListener**: Intercepts every controller invocation. If `SITE_MODE=extranet`, denies access unless the user has `ROLE_WEB_ACCOUNT` or the controller action has an explicit `@IsGranted`/`@Security` attribute.

**GraphQlDefaultAccessConfigProcessor**: Runs at priority 1024 during GraphQL schema processing. Injects a default access-denial expression on all RootQuery and RootMutation fields in extranet mode when the user lacks `ROLE_WEB_ACCOUNT`.

## Code Standards

- PHP 8.1–8.4, Symfony 6.3+ or 7.0+
- PHPStan level 9 — all code in `src/` must pass strict analysis
- Code style: PER-CS standard enforced by PHP-CS-Fixer
- Conventional Commits format enforced by `.githooks/commit-msg`
- Tests live in `test/` (PSR-4 namespace `Atoolo\Extranet\Test\`)
- Test tools managed via PHIVE in `tools/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sitepark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
