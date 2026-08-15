## osmosisinthetrees

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Website for the Osmosis in the Trees music festival. Built on **Statamic 5** (flat-file CMS on Laravel 11) using the **Peak starter kit** from Studio 1902. Frontend is Antlers templates + Alpine.js + Tailwind CSS v3, bundled with Vite.

## Commands

```bash
# Install / setup
composer install
npm install
php please make:user           # create a CP user

# Develop
npm run dev                    # Vite dev server with HMR (refresh on file changes)
php artisan serve              # Laravel dev server (optional; typically use Valet/Herd)

# Build for production
npm run build

# Statamic / Laravel CLIs
php please <command>           # Statamic's CLI (wraps artisan + statamic commands)
php artisan <command>          # raw Laravel
php please stache:warm         # rebuild Statamic's content cache
php please static:clear && php please static:warm   # regen full static cache

# Tests (PHPUnit)
./vendor/bin/phpunit                                # all suites
./vendor/bin/phpunit --testsuite=Feature            # Feature suite only
./vendor/bin/phpunit --filter=SomeTest              # single test

# Code style
./vendor/bin/pint                                   # Laravel Pint (PHP formatter)
```

## Architecture

### Content is in `content/`, not the database

Statamic is flat-file — all editable content lives as YAML/Markdown on disk. SQLite (`database/database.sqlite`) is used only for Laravel's framework tables (sessions, cache, jobs). Do not look in the DB for site content.

- `content/collections/{artists,pages,information}/` — entries as `.md` files
- `content/collections/*.yaml` — collection-level config (sort, dates, routes, mounts)
- `content/globals/` — global sets: `configuration`, `header`, `seo`, `social_media`, `browser_appearance`, `redirects`
- `content/trees/` — navigation + collection ordering
- `content/taxonomies/`, `content/assets/` — taxonomies and asset container configs
- `users/` — CP user accounts as YAML

### Schema lives in `resources/blueprints/` and `resources/fieldsets/`

- **Blueprints** (`resources/blueprints/collections/{artists,pages,information}/…`) define the fields for each entry type.
- **Fieldsets** (`resources/fieldsets/*.yaml`) are reusable field groups. The key one is `page_builder.yaml`, which powers a Bard/Replicator-style page-builder field.

### Page Builder pattern (Peak convention)

Pages compose layout blocks via a page-builder field. Each block type has:
1. A fieldset in `resources/fieldsets/` (e.g. `article.yaml`, `cards.yaml`).
2. A matching partial in `resources/views/page_builder/_<type>.antlers.html` rendered by the page_builder loop.

When adding a new block: add the fieldset, reference it in `resources/fieldsets/page_builder.yaml`, and create the matching `_<type>.antlers.html` partial.

### Templates / views

- `resources/views/layout.antlers.html` — master layout (header, footer, Vite tags, SEO/browser-appearance partials from Peak).
- `resources/views/default.antlers.html` — default entry template.
- `resources/views/layout/`, `resources/views/components/`, `resources/views/page_builder/`, `resources/views/navigation/` — reusable partials (leading `_` = partial, referenced without the underscore via `{{ partial:name }}`).
- Peak ships partials under `vendor/studio1902/statamic-peak-*` — these are scanned by Tailwind's `content` config, so classes used inside vendor partials are preserved in the build.

### Frontend build

- Vite entry points are declared in `vite.config.js` and loaded via `{{ vite src="…" }}` in Antlers. The three main bundles are:
  - `resources/css/site.css` + `resources/js/site.js` (core)
  - `resources/js/effects/text-animation/text-animation-1.js`
  - `resources/js/effects/svg-filter/main.js` (+ `filters/filter1.js`)
- Tailwind config is split across presets: `tailwind.config.js` (root) → `typography`, `peak`, `site`. Project-specific theming (colors like `osmosis-brown`, `osmosis-orange`, `highlight-yellow`, `primary`) lives in `tailwind.config.site.js`.
- `@` alias resolves to `resources/` for JS imports.

### Laravel-side code (`app/`)

The `app/` directory is intentionally thin — this is a CMS-driven site, not an app with heavy controllers. What's there:

- `app/Listeners/PreventDeletingMounts.php` — blocks deleting a page entry that is mounted by a collection (e.g. the `/artists` page mounting the artists collection). If you move or rename a mounted page, update the collection's `mount:` in its YAML first.
- `app/Http/Controllers/` — only the abstract base `Controller.php`; routes in `routes/web.php` are intentionally empty because Statamic handles routing through collections + `Route::statamic()`.

### Static caching (production)

Production uses Statamic's **full static caching** (`STATAMIC_STATIC_CACHING_STRATEGY=full`). After deploys, the stache is warmed, search is re-indexed, and static cache is cleared + re-warmed (see README deploy scripts). Locally, static caching is usually off — if pages seem stale in production, the static cache is the suspect, not Laravel's route/config cache.

### Peak plugins in use

`studio1902/statamic-peak-seo`, `statamic-peak-browser-appearance`, `statamic-peak-tools`. They inject partials for SEO meta, live preview, noscript, skip-to-content, and browser appearance (favicons/theme-color). Editing SEO fields goes through the `seo` global + per-entry seo fields from those packages — don't add your own `<meta>` tags in `layout.antlers.html` unless you're doing something the plugin can't.

## Conventions & gotchas

- `public/images/artists/` is git-ignored — it's generated from asset sources; don't commit it.
- The deploy scripts short-circuit when the commit message contains `[BOT]` (used for Statamic's Git-driven control-panel commits). If you push a real change with `[BOT]` in the message it won't deploy.
- `STATAMIC_GIT_ENABLED=true` + `STATAMIC_GIT_PUSH=true` in prod: editors saving in the CP auto-commit + push content changes. Keep this in mind — `git pull` before local content edits or you'll conflict.
- `routes/web.php` is intentionally empty. Add routes there only for genuinely non-Statamic endpoints; otherwise let collections define URLs via their `route:` key.

---
> Source: [PhilippeGauthier/osmosisinthetrees](https://github.com/PhilippeGauthier/osmosisinthetrees) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
