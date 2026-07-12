## studentenportal

> Read the repository-root [`AGENTS.md`](../AGENTS.md) before making changes. It is the primary, current operating guide for all AI agents in this repository. Do not maintain a second copy of stack, architecture, endpoint, command, or testing facts in this file.

# Copilot Project Instructions - CampusConnect

Read the repository-root [`AGENTS.md`](../AGENTS.md) before making changes. It is the primary, current operating guide for all AI agents in this repository. Do not maintain a second copy of stack, architecture, endpoint, command, or testing facts in this file.

## Repository Entry Points

- Main application: `CampusConnect/`
- Product scope: `CampusConnect/docs/product/projektbeschreibung.md`
- Setup and project overview: `CampusConnect/docs/project-overview.md`
- Architecture and authentication: `CampusConnect/docs/architecture.md`
- Implemented API behavior: `CampusConnect/docs/api.md`
- Testing conventions: `CampusConnect/docs/testing.md`
- Contribution workflow: `CampusConnect/docs/contributing.md`

When documentation conflicts with code or configuration, use the live implementation for code changes and update the affected documentation in the same change.

## Copilot Checklist

1. Read `AGENTS.md`, then the nearest source files and tests.
2. Keep changes focused and preserve existing architecture boundaries.
3. Do not put persistence or business logic in controllers or Angular components.
4. Do not call external SWFR or DHBW sources from the frontend.
5. Keep authentication tokens in memory only; never store them in `localStorage` or `sessionStorage`.
6. For user-facing frontend text, add English and German keys in `CampusConnect/frontend/src/app/core/i18n/translations.ts` and use `TranslatePipe` or the `I18n` service.
7. Update API, architecture, setup, or testing docs whenever related behavior changes.
8. Run the smallest relevant tests and build, and report anything that could not be validated.

## Current Infrastructure Status

- GitHub Actions CI is implemented in `CampusConnect/.github/workflows/ci.yml` and runs backend restore/build/test plus frontend install/test/build.
- `CampusConnect/docker-compose.yml` is a placeholder and is not a supported production deployment.

---
> Source: [VEGA-DH-Informatik-25/StudentenPortal](https://github.com/VEGA-DH-Informatik-25/StudentenPortal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
