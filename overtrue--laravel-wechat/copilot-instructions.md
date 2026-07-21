## laravel-wechat

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

This repository is maintained by AI assistance (OpenClaw).

## Goal

- Support **Laravel 13+ only**
- Minimum PHP: **8.3**
- Keep a clean CI signal (tests + lint/format) before tagging a release

## Local workflow

```bash
composer install
composer check-style
```

## Release workflow

1. Ensure `composer.json` supports Laravel 13 / PHP 8.3.
2. Run code quality checks (Pint) and any test suite (if present).
3. Commit changes.
4. Wait for GitHub Actions to pass.
5. Tag & publish release using `gh release create`.

## Notes

- This package is a Laravel wrapper for `w7corp/easywechat`.
- If Laravel introduces contract signature changes, prefer updating the wrapper to match Laravel 13 APIs.

---
> Source: [overtrue/laravel-wechat](https://github.com/overtrue/laravel-wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
