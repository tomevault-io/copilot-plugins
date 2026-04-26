## vcul-staff-directory-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WordPress plugin for managing staff directories at VCU Libraries (VCUL). It provides a custom post type, REST API, and Vue 3 frontend for both public and internal staff directory views.

## Commands

### JavaScript/CSS Build

```bash
npm install

# Development (watch mode)
npm run start

# Production build
npm run build
```

The build entry point is `assets/src/index.js`, outputting to `assets/dist/`. The legacy Grunt-based build also exists (`npm run build:old`), but prefer `wp-scripts`.

### PHP Dependencies

```bash
composer install    # Installs dev deps and configures phpcs with WPCS
composer update
```

### Linting

```bash
# PHP (WordPress Coding Standards)
./vendor/bin/phpcs

# CSS
npx stylelint "**/*.css"

# JS (via Grunt)
npx grunt jshint
npx grunt jscs
```

### Tests

```bash
# PHPUnit (requires WordPress test environment)
./vendor/bin/phpunit

# With coverage (requires xdebug)
./vendor/bin/phpunit --coverage-clover coverage.xml
```

PHPUnit configuration is in `phpunit.xml`; bootstrap is `tests/bootstrap.php`.

## Architecture

### Entry Point

`plugin.php` — Defines plugin constants (`VCUL_DIRECTORY_PATH`, `VCUL_DIRECTORY_URL`, version, etc.) and the `bootstrap()` function that requires all includes. Uses the `VCUL\Directory` namespace.

### Backend (`/includes`)

| File | Responsibility |
|------|----------------|
| `directory-post-type.php` | Registers `directory` CPT and `expertise`, `department`, `specialist` taxonomies; defines all post meta fields (email, phone, address, pronouns, rank, title, CV, LibCal, guides); handles sanitization |
| `rest-api.php` | REST API class (`Rest_API`) under `/wp-json/vcul-directory/v1/`; endpoints: `get_experts()`, `get_department()`, `get_settings()`; enforces privacy rules |
| `directory-shortcodes.php` | Registers `[vcul_directory]` shortcode; enqueues assets; renders `<div id="app">` as Vue mount point |
| `directory-contributor-role.php` | Creates `vcul_directory_contributor` role with scoped capabilities; limits media library to own uploads |
| `directory-settings.php` | Admin settings page (org chart PDF URL) under the Staff Directory menu |

### REST API Privacy Model

`rest-api.php` checks the `HTTP_SEC_FETCH_SITE` header to distinguish same-origin vs. external requests. Phone numbers and profile photos can be hidden for external/anonymous requests, controlled by per-entry post meta (`directory_hide_phone`, `directory_hide_photo`).

### Frontend (`/js`, `/css`, `/assets`)

- `js/directory.min.js` — Compiled Vue 3 app (do not edit directly)
- `css/directory.css` — Compiled frontend styles (do not edit directly)
- `css/_variables.scss` — SCSS design tokens and variables (source of truth for styles)
- `assets/src/index.js` — Source entry point built via `@wordpress/scripts`
- `js/directory-media.js` — Admin media uploader helper for headshot uploads

### Data Model

Each staff member is a `directory` post with these key meta fields:
- Contact: `directory_email`, `directory_phone`, `directory_address`
- Identity: `directory_pronouns`, `directory_rank`, `directory_professional_title`
- Links: `directory_cv_link`, `directory_libcal_link`, `directory_guides` (serialized array)
- Privacy: `directory_hide_phone`, `directory_hide_photo`
- Taxonomy: `expertise` (multi), `department` (single), `specialist` (multi)

### CI

GitHub Actions workflows in `.github/workflows/`:
- `php.yml` — Validates composer on push/PR
- `ci-coverage.yml` — Runs PHPUnit with xdebug coverage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/streeetlamp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
