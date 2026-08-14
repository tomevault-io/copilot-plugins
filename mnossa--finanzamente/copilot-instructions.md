## finanzamente

> > **Token-optimized layout.** Always-on rules: `.cursor/rules/core.mdc` + `agent-workflow.mdc`.

# Finanzamente — Cursor rules index

> **Token-optimized layout.** Always-on rules: `.cursor/rules/core.mdc` + `agent-workflow.mdc`.
> File-scoped rules load by glob (PHP, React, Blade, E2E, GDPR).
> Long reference: `docs/agent/`. Cloud VMs: `AGENTS.md`.

## Output (always)
Default: **caveman full**. Off: `stop caveman` | `normal mode`. `/caveman lite|full|ultra`. Critical security: clear text first.

## Quick reference
| Topic | Where |
|-------|--------|
| Stack, naming, locale | `.cursor/rules/core.mdc` |
| Gates: test → pint → playwright | `.cursor/rules/agent-workflow.mdc` |
| Laravel / PHP | `.cursor/rules/php-laravel.mdc` |
| React / Inertia | `.cursor/rules/react-inertia.mdc` |
| Blade public | `.cursor/rules/blade-public.mdc` |
| Playwright | `.cursor/rules/e2e-playwright.mdc` |
| Privacy / GDPR | `.cursor/rules/gdpr-privacy.mdc` |
| Make commands | `docs/agent/makefile.md` |
| E2E detail | `docs/agent/e2e-conventions.md` |
| PWA, ops, extensibility | `docs/agent/architecture.md` |

## Non-negotiables
- Docker + `make *` only for dev commands.
- UI Italian; code/DB English.
- MIT open source, self-host — no SaaS billing / plan tiers.
- New feature → tests; runtime change → `make test`, `make pint-check`, `make playwright` (see workflow rule).
- Privacy policy change → version in `config/legal.php` + consent alignment + tests.

Update `.cursor/rules/*.mdc` when conventions change; keep this file as a short index.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
