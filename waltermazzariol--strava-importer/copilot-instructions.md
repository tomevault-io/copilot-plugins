## strava-importer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

WordPress plugin that imports Strava activities as WordPress posts via Strava API v3. Single-class PHP plugin with no build system, no external PHP dependencies, and no test framework. Ready to use as-is in WordPress.

**Requirements**: WordPress 5.0+, PHP 7.4+

## Architecture

**Single class**: `Strava_Activity_Importer` (singleton pattern) in `strava-activity-importer.php` (~770 lines). All plugin logic lives here.

**File layout**:
- `strava-activity-importer.php` — Main plugin file (class, hooks, API, import logic)
- `templates/admin-page.php` — Admin page HTML template
- `assets/admin.js` — jQuery-based admin interactions (AJAX, pagination, import progress)
- `assets/admin.css` — Admin UI styling with Strava brand colors

### Key flows

**OAuth**: Settings page → "Connect with Strava" → OAuth redirect → `handle_oauth_callback()` exchanges code for tokens → stored as WP options (`strava_access_token`, `strava_refresh_token`, `strava_token_expires_at`). Token auto-refreshes 60s before expiry in `get_access_token()`.

**Import**: `ajax_fetch_activities()` fetches paginated activity list → user selects activities → `ajax_import_activity()` runs per-activity:
1. Fetches activity details from `activities/{id}`
2. Fetches photos from `activities/{id}/photos?size=2048`
3. Builds block-editor HTML content via `build_post_content()`
4. Creates WP post with `wp_insert_post()`
5. Downloads all photos to media library via `download_all_photos()` → `download_photo()`
6. Sets first photo as featured image, rebuilds content with local image URLs

**Post content**: Generated as WordPress block markup (`<!-- wp:paragraph -->`, `<!-- wp:gallery -->`, etc.), not shortcodes. Includes stats table, photo gallery, and Strava link.

### AJAX endpoints (all require `manage_options`)

- `strava_fetch_activities` — List activities with duplicate detection
- `strava_import_activity` — Import single activity as post
- `strava_disconnect` — Clear stored credentials

### Data storage

- **Options**: OAuth tokens, client credentials, default post status/author, athlete info
- **Post meta**: All prefixed `_strava_*` (activity_id, distance, speed, heartrate, polyline, etc.)
- **Categories**: Auto-creates "Strava Activities" parent + sport-type subcategories

## Conventions

- WordPress coding standards (tabs, `wp_*` functions, hooks)
- All AJAX: nonce verification + capability check
- Input: `sanitize_text_field()`, `absint()`, `sanitize_file_name()`
- Output: `esc_html()`, `esc_url()`, `esc_attr()`
- Text domain: `strava-importer` (i18n ready with `__()` / `_e()`)
- JS: jQuery-based, ES5 (no transpilation), 500ms delay between sequential imports
- Photos downloaded to WP media library via `download_url()` + `media_handle_sideload()`
- When making changes, please update the CHANGELOG.md file under a 'Unreleased' section, following Keep a Changelog formatting.
- Upgrade the version of the plugin

---
> Source: [waltermazzariol/strava-importer](https://github.com/waltermazzariol/strava-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
