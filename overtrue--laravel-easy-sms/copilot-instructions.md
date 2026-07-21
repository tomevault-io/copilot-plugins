## laravel-easy-sms

> - Prefer minimal, high-confidence changes.

# AGENTS.md

## Maintenance guidelines

- Prefer minimal, high-confidence changes.
- Do not change public APIs unless the issue explicitly requires it.
- Do not introduce new dependencies without a clear reason.
- Keep diffs small and reviewable.
- For PHP projects, check composer scripts first; prefer the smallest relevant PHPUnit/Pest/PHPStan command.
- For JavaScript/TypeScript projects, check package scripts first; prefer targeted tests/lint/typecheck.
- Never merge PRs, publish releases, close controversial issues, or modify security policy automatically.

## Review guidelines

- Flag regressions, missing tests, BC breaks, security risks, and unclear behavior.
- Do not block on subjective style unless it violates existing project conventions.
- Treat documentation typos as low priority unless they change meaning.
- When suggesting changes, be specific and include the reason.

Package: **overtrue/laravel-easy-sms**

## Purpose
Laravel integration (service provider) for **overtrue/easy-sms**.

## Compatibility Targets
- Laravel: **^13**
- PHP: **^8.3**

## Local Development
```bash
composer install
composer test
composer check-style
```

## CI / Release
- Do not tag / release unless GitHub Actions on `master` is green.

---
> Source: [overtrue/laravel-easy-sms](https://github.com/overtrue/laravel-easy-sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
