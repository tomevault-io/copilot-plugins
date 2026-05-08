## ddev-drupal-admin-vrt

> DDEV add-on that provides visual regression testing for Drupal's `default_admin` admin theme using Playwright. Screenshots admin pages at 3 viewports (narrow/mid/wide) and compares against baselines.

# CLAUDE.md

## Project overview

DDEV add-on that provides visual regression testing for Drupal's `default_admin` admin theme using Playwright. Screenshots admin pages at 3 viewports (narrow/mid/wide) and compares against baselines.

## Key files

- `install.yaml` — DDEV add-on manifest. `project_files` must be flat strings, not source/destination maps.
- `drupal-admin-vrt/playwright.config.ts` — Playwright config. Defines 4 projects: `auth-setup`, `narrow`, `mid`, `wide`. Snapshots go to `../../__screenshots__/`, test results to `../../test-results/`, reports to `../../playwright-report/` (all relative to the drupal-admin-vrt dir, landing in the Drupal project root).
- `drupal-admin-vrt/fixtures/auth.setup.ts` — Logs in via `drush uli --uid=1` (no `--no-browser` flag, it doesn't exist in Drupal 12's drush). Falls back to form login when `DRUPAL_ADMIN_USER`/`DRUPAL_ADMIN_PASS` env vars are set.
- `drupal-admin-vrt/page-definitions/admin-pages.ts` — Central registry of pages to test. Each entry has `id`, `path`, `section`, and optional `fullPage`, `waitFor`, `maskSelectors`, `timeout`, `interactions`.
- `drupal-admin-vrt/page-definitions/theming-tools-pages.ts` — Page definitions for the optional theming_tools contrib module. Tests auto-skip when a submodule's route returns 404.
- `drupal-admin-vrt/tests/vrt/generate-vrt-tests.ts` — Shared test generator consumed by per-section spec files.
- `drupal-admin-vrt/tests/vrt/theming-tools.spec.ts` — Theming tools tests with per-test 404 skip logic (doesn't use the shared generator since it needs custom skip behavior).
- `commands/web/vrt`, `commands/web/vrt-update`, `commands/web/vrt-report` — DDEV custom commands.

## Testing locally

The add-on files live in `ddev-drupal-admin-vrt/` but need to be copied into a Drupal project's `.ddev/` directory to test. During development:

```bash
# Copy files into the Drupal project
cp -r commands/web/* /path/to/drupal/.ddev/commands/web/
cp -r drupal-admin-vrt /path/to/drupal/.ddev/drupal-admin-vrt
cp docker-compose.vrt-report.yaml /path/to/drupal/.ddev/

# Or install from local path
ddev add-on install /path/to/ddev-drupal-admin-vrt
```

The Drupal test site is at `../drupal/` relative to this repo.

## Common pitfalls

- `install.yaml` `project_files` entries must be plain strings, not `source`/`destination` objects.
- The `vrt-report` command must point to `../../playwright-report` since that's where the config outputs it.
- The `vrt-update` command puts `--update-snapshots` at the end so file path args can be passed before it: `npx playwright test "$@" --update-snapshots`.
- Large pages (like permissions) need a custom `timeout` in the page definition for screenshot stability.
- `hide-dynamic.css` suppresses timestamps, tokens, and other non-deterministic content to prevent false positives.
- Playwright runs inside the DDEV web container; the base URL is `https://localhost` (not the `.ddev.site` hostname).
- The DDEV commands run in the web container (`commands/web/`), not on the host.

## Publishing

Install URL: `ddev add-on install https://github.com/mherchel/ddev-drupal-admin-vrt/tarball/main`

The bare repo URL (`https://github.com/mherchel/ddev-drupal-admin-vrt`) does not work with `ddev add-on install` — the `/tarball/main` suffix is required.

---
> Source: [mherchel/ddev-drupal-admin-vrt](https://github.com/mherchel/ddev-drupal-admin-vrt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
