## kirby-mcp

> Keep GitHub Actions fast, reproducible, and aligned with local `composer` scripts.

# CI Workflow Guidelines

## Mission

Keep GitHub Actions fast, reproducible, and aligned with local `composer` scripts.

## System

- `pest-tests.yml` runs Pest on PHP 8.5 (with Xdebug coverage).
- `phpstan.yml` runs `composer analyse` (PHPStan on PHP 8.5 with the autoload helper fix) and uses `--error-format=github`.
- `fix-php-code-style-issues.yml` runs Laravel Pint and auto-commits styling fixes.

## Workflows

- If you change PHP support or `composer.json` scripts (`test`, `analyse`, `format`), update workflows to match.
- Install steps in CI currently run `composer install`; keep this in sync with how deps are installed for CI.

## Guardrails

- Avoid auto-commit loops (Pint action + auto-commit).
- Prefer `github.ref` on push workflows and guard with `if: github.actor != 'github-actions[bot]'`.
- Keep secrets out of logs; pin action versions.
- Don’t add network-dependent tests or long-running steps.

---
> Source: [bnomei/kirby-mcp](https://github.com/bnomei/kirby-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
