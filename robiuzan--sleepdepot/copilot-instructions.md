## sleepdepot

> WooCommerce + ACF Pro store on LocalWP. This file is the source of truth for **this** site.

# sleepdepot (sleepdepot.co.il) — project instructions

WooCommerce + ACF Pro store on LocalWP. This file is the source of truth for **this** site.
WP/Woo/ACF engineering rules for this project live locally (no global config) — imported here:

@.claude/wordpress-base.md

## Identity
- **Store**: sleepdepot.co.il (Israeli ecommerce store — mattresses/sleep products)
- **LocalWP site name**: `sleepdepot` · **Local domain**: https://sleepdepot.local · **Live**: https://sleepdepot.co.il
- **WordPress**: 7.0 · table prefix `wp_`
- **Locale**: Hebrew/Israel → **RTL** and i18n matter (see `/sleepdepot-hebrew-rtl`, `/i18n-l10n`, `/responsive-design`). Confirm the WP locale is `he_IL`.
- Prefix for new custom code: `sleepdepot_` · Text domain: `sleepdepot`

## ⚠ IMPORTANT — code location vs. working dir
- This Claude project dir is `C:\Users\robiu\antigravity\Projects\sleepdepot.co.il` (holds `.claude/` + this file). It is the env that is version-controlled in GitHub (`robiuzan/sleepdepot`).
- **The actual WordPress install is elsewhere** (LocalWP default location):
  `C:\Users\robiu\Local Sites\sleepdepot\app\public`
- Always edit theme/plugin files at that **absolute path** (it's wired as an additional working dir in `.claude/settings.json`). Consider opening `C:\Users\robiu\Local Sites\sleepdepot` as the workspace.

## Theme Architecture: **Classic**
- **Active theme: `sleepdepot`** — a standalone **HTML5 Blank**–based theme (classic PHP templates + hooks, not FSE → `/theme-json-architect` does **not** apply). **No parent/child** relationship; it is self-contained.
- Theme path: `...\app\public\wp-content\themes\sleepdepot\`
- Modular layout: `functions.php` only bootstraps; real logic lives under `settings/` (`acf/`, `enqueue_scripts.php`, `enqueue_styles.php`, `general.php`, `additional/`, `functions/`). Front-end page content is **ACF/DB-driven** via a page-builder (`templates/page-builder.php` + `parts/sections/section-*.php`). Full map in `/sleepdepot-theme`.
- WooCommerce overrides → `sleepdepot\woocommerce\...` (never edit the Woo plugin). Prefer `woocommerce_*` hooks/filters over template edits.
- Other themes present (`genesis-block-theme`, `twentytwenty*`) are **inactive** — editing them has no front-end effect.

## Stack / key plugins (installed)
- **Commerce**: WooCommerce + Product Bundles, Side Cart, Woo Discount Rules (+ Pro), **WebToffee Smart Coupons Pro** (`wt-smart-coupon-pro`), Advanced Free Shipping, WPC variation bulk editor, order + customer import/export. See `/sleepdepot-promotions`.
- **Payments (Israel)**: **iCredit** (`woocommerce-icredit 4.0.4` — note the space in the folder name) + PayPal Payments (secondary). Treat orders via Woo CRUD/HPOS API. See `/sleepdepot-icredit`. (No Rivhit/invoicing plugin here.)
- **Fields**: **ACF Pro** — local JSON sync is **already wired** to `themes/sleepdepot/settings/acf/acf-json/`. See `/wp-acf`.
- **Forms**: none active. Contact Form 7 is scaffolded in the theme but its include is commented out and the plugin isn't installed.
- **SEO**: Rank Math (`seo-by-rank-math`) — owns titles/meta/schema/sitemaps; don't duplicate. See `/seo-technical`, `/seo-onpage-schema`.
- **Performance/optim**: **Autoptimize** (cache/minify — no WP Rocket), `webp`/media via Enable Media Replace. See `/wp-performance`, `/core-web-vitals`, `/images-media`.
- **Blocks/UI**: Genesis Blocks + a theme-local Gutenberg `cta-block`; Popup Maker; SVG Support; Creame WhatsApp-me.
- **Security**: Wordfence, WPS Hide Login, Force Strong Passwords. See `/security-hardening`.
- **Headers/scripts**: Insert Headers and Footers (where GTM/analytics snippets would live). See `/analytics-tracking`.
- **Migration/backup**: All-in-One WP Migration (+ Pro), UpdraftPlus, BlogVault/Cloudways automated migration, WPE site migration. See `/sleepdepot-deploy`.
- **Other**: Jetpack, Akismet, WP File Manager.
- ⚠ **Mixed host origin**: mu-plugins from **Hostinger**, **WP Engine**, and **ManageWP/BlogVault** are all present — this site has been migrated more than once. Confirm the current live host before any deploy (see `/sleepdepot-deploy`).

## ⚠ Security note (theme ACF config)
`themes/sleepdepot/settings/acf/acf-config.php` enables `acf/shortcode/allow_unsafe_html` + `acf/the_field/allow_unsafe_html` and runs `do_shortcode()` over ACF values. **ACF output is not auto-escaped here** — escape on output in templates yourself, and restrict who can embed shortcodes. Worth a `security-auditor` pass. See `/security-hardening`, `/wp-standards`.

## Tooling
- **wp-cli**: open the **LocalWP site shell** for `sleepdepot` (so `wp` targets this DB). wp-cli is NOT on the system PATH — only inside that shell.
- **Debug logging**: ✅ enabled in `wp-config.php` (2026-06-10) — `WP_DEBUG`/`WP_DEBUG_LOG`/`SCRIPT_DEBUG` on, `WP_DEBUG_DISPLAY` off + `display_errors=0`. Errors log to `wp-content/debug.log` (created lazily on first notice). See `/wp-debug`.
- **Pulse** after PHP edits (reads the real site's debug.log):
  `powershell -NoProfile -File C:\Users\robiu\antigravity\Projects\sleepdepot.co.il\.claude\hooks\tail-wp-logs.ps1 -SitePath "C:\Users\robiu\Local Sites\sleepdepot"`
- **Lint**: add a `phpcs.xml` to the theme (pragmatic WPCS, errors-only) to activate the `phpcs-on-save` hook, then `composer require --dev wp-coding-standards/wpcs dealerdirect/phpcodesniffer-composer-installer` in the theme dir. See `/wp-standards`.
- **Snapshot before risk**: `wp db export` (the `db-export` hook auto-snapshots before risky CLI ops when `wp` is on PATH).

## Hooks (in `.claude/hooks/`, wired in `.claude/settings.json`)
- `guard-wp-core` (PreToolUse Edit|Write) — blocks edits to `wp-admin/`, `wp-includes/`, `vendor/`, `node_modules/`, core files.
- `guard-db-commands` + `db-export` (PreToolUse Bash|PowerShell) — block destructive DB ops; auto-snapshot before risky ones.
- `phpcs-on-save` + `acf-json-reminder` (PostToolUse Edit|Write) — lint PHP against a ruleset if present; remind to keep `acf-json/` in sync.

## Skills & agents for this site
**Project-specific skills** (live in `sleepdepot.co.il\.claude\skills\`, sleepdepot-only):
- `/sleepdepot-theme` — the `sleepdepot` theme: modular `settings/`, enqueue handles, ACF page-builder sections, Gutenberg CTA block, Woo overrides.
- `/sleepdepot-icredit` — iCredit (Israeli payments) + the `$zones` bug fix (still in 4.0.4); PayPal secondary.
- `/sleepdepot-promotions` — Discount Rules + Smart Coupons + Bundles + Advanced Free Shipping + Side Cart interaction.
- `/sleepdepot-deploy` — env + migration/deploy workflow (mixed Hostinger/WPE/BlogVault origin).
- `/sleepdepot-hebrew-rtl` — Hebrew/RTL/ILS conventions (RTL is the default here; copy is ACF/DB-driven).

**WordPress toolkit skills** (local in `.claude/skills/`): `/wp-acf`, `/woocommerce-dev`, `/woo-data-mocking`, `/wp-standards`, `/wp-debug`, `/frontend-build`, `/wp-performance`, `/core-web-vitals`, `/images-media`, `/seo-technical`, `/seo-onpage-schema`, `/analytics-tracking`, `/responsive-design`, `/accessibility`, `/design-system`, `/conversion-cro`, `/content-modeling`, `/copy-microcopy`, `/i18n-l10n`, `/security-hardening`, `/backup-deploy`, `/testing-qa`, `/forminator-dev`.
(Not applicable here: `/forminator-dev` — no Forminator installed; kept as a generic reference only.)

**Audit agents** (in `.claude/agents/`): `wp-reviewer`, `acf-architect`, `seo-auditor`, `a11y-auditor`, `performance-auditor`, `responsive-ux-auditor`, `conversion-optimizer`, `content-strategist`, `security-auditor`.

## CPT / Taxonomy / Field-group registry (living list — keep in sync with memory)
- CPTs: _TODO (detect via wp-cli: `wp post-type list`)_
- Taxonomies: _TODO (`wp taxonomy list`)_
- ACF field groups: _TODO — synced JSON lives in `themes/sleepdepot/settings/acf/acf-json/`; list the page-builder section layouts here as you map them._

---
> Source: [robiuzan/sleepdepot](https://github.com/robiuzan/sleepdepot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
