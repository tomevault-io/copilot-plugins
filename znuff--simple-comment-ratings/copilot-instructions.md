## simple-comment-ratings

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Simple Comment Ratings** — a WordPress plugin. Visitors (including non-logged-in) can vote up/down on comments. One vote per IP per comment, no take-back. Cloudflare Turnstile for bot protection.

- PHP minimum: 7.4 | WordPress minimum: 5.8
- No build step — plain PHP, CSS, and vanilla JS
- Text domain: `simple-comment-ratings` | Global prefix: `scr_` / `SCR_`

## File Structure

```
simple-comment-ratings.php          Entry point: constants, requires, activation hook
uninstall.php                       Drops both DB tables + options on uninstall
includes/
  class-scr-database.php            All DB queries (two-table design, see below)
  class-scr-settings.php            WP Settings API wrapper; singleton
  class-scr-renderer.php            HTML generation for vote widget; static post-level cache
  class-scr-ajax.php                AJAX handlers: scr_vote, scr_get_counts
  class-scr-plugin.php              Hook registration; icon set filter; asset enqueueing
assets/
  css/simple-comment-ratings.css
  js/simple-comment-ratings.js
languages/                          .pot file lives here
```

## Database Design (Two-Table)

**`wp_comment_ratings`** — audit log, one row per IP+comment.
- Unique key `(comment_id, ip_address)` enforces one-vote-per-IP at the DB level.
- Used only for: duplicate-vote checks and IP-level user-vote lookups.

**`wp_comment_rating_counts`** — denormalized summary, PK on `comment_id`.
- Updated atomically via `INSERT … ON DUPLICATE KEY UPDATE` on every vote.
- All count reads are PK lookups — no `GROUP BY` at read time regardless of vote volume.

### Key queries

Counts + user votes for an entire post use a single subquery each (never a PHP-built ID list):
```sql
SELECT crc.comment_id, crc.up_count, crc.down_count
FROM   wp_comment_rating_counts crc
WHERE  crc.comment_id IN (
    SELECT comment_ID FROM wp_comments
    WHERE  comment_post_ID = %d AND comment_approved = '1'
)
```

## Renderer Cache

`SCR_Renderer::render()` is called once per comment (via `comment_text` filter). On the **first** call for a given post the renderer loads all counts and the current IP's votes for that post in two queries, stored in static arrays. All subsequent comments on the same post hit no DB at all.

## Turnstile

A plain `<div id="scr-turnstile-container">` (no `cf-turnstile` class) is output in `wp_footer`. The script URL includes `?render=explicit&onload=scrOnTurnstileLoad` — `render=explicit` disables Turnstile's Intersection Observer (which would otherwise delay the challenge until the element scrolls into view). `window.scrOnTurnstileLoad` calls `turnstile.render()` with `execution: 'render'` (challenge starts immediately) and `appearance: 'interaction-only'` (UI only appears if Cloudflare requires user interaction). Token is stored in the JS closure; pending votes queue until it arrives. `consumeToken()` calls `turnstile.reset(widgetId)` after each vote.

If no Turnstile sitekey is configured the widget and script are never loaded; the AJAX handler skips verification.

## Settings

All stored under `scr_options` (single serialized option). Keys:

| Key | Default | Values |
|-----|---------|--------|
| `turnstile_sitekey` | `''` | string |
| `turnstile_secret`  | `''` | string |
| `display_position`  | `bottom-right` | `bottom-right` / `bottom-left` / `top-right` / `top-left` |
| `display_style`     | `separate` | `separate` (count next to each button) / `sum` (net score between buttons) |
| `icon_set`          | `fontawesome` | any key registered via `scr_icon_sets` filter |
| `counts_loading`    | `server_side` | `server_side` / `ajax` |

## Extensibility

**Adding an icon set** — hook `scr_icon_sets` (priority < 10 to appear in settings dropdown):
```php
add_filter( 'scr_icon_sets', function ( $sets ) {
    $sets['arrows'] = [
        'label' => 'Arrows',
        'up'    => '<i class="fa-solid fa-arrow-up" aria-hidden="true"></i>',
        'down'  => '<i class="fa-solid fa-arrow-down" aria-hidden="true"></i>',
    ];
    return $sets;
} );
```

**Overriding the Font Awesome CDN URL** — filter `scr_font_awesome_url`.

## AJAX Endpoints

Both accept `wp_ajax_nopriv_*` (unauthenticated users).

| Action | Method | Key inputs | Returns |
|--------|--------|------------|---------|
| `scr_vote` | POST | `comment_id`, `vote` (up/down), `nonce`, `turnstile_token` | `{ up, down, sum, user_vote }` |
| `scr_get_counts` | POST | `post_id`, `nonce` | `{ [comment_id]: { up, down, sum, user_vote } }` |

`scr_get_counts` is only called when `counts_loading = ajax`; PHP passes `scrData.postId` to JS via `wp_localize_script`.

## JS Data (`scrData`)

`wp_localize_script` provides:
- `ajaxUrl`, `nonce`, `sitekey`, `countsLoading`, `postId`, `i18n`

## CSS Position System

The vote container uses `position: absolute` relative to `.comment-body` / `.comment-content` (both get `position: relative` via CSS). Position class on the container: `scr-pos-bottom-right`, `scr-pos-bottom-left`, `scr-pos-top-right`, `scr-pos-top-left`. A `padding-bottom: 2.25rem` on `.comment-content` reserves space so the bar never overlaps text.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Znuff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
