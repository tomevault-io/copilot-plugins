## consentric

> **Human = Product Owner. AI = Developer.**

# Consentric — CLAUDE.md

## Working Agreement

**Human = Product Owner. AI = Developer.**

- All requirements are broken into discrete features/tasks.
- Before implementing any task, the AI presents scope + acceptance criteria and waits for PO approval.
- Each approved feature gets its own git branch (`feature/feat-XX-slug`), merged to `main` when complete.
- No code is written speculatively — only what has been approved.
- Never include FEAT-XX IDs or branch names in admin/frontend UI text.
- Always show the implementation and wait for PO approval before committing.

---

## Slash Commands (Skills)

Use these when the PO makes the corresponding request:

| Command | When to use |
|---------|-------------|
| `/new-feature FEAT-XX description` | PO asks to implement a new feature |
| `/release X.X.X` | PO asks to release a new version |
| `/translate` | New strings were added; translations need updating |
| `/add-setting option_name tab field_type` | PO asks to add a new admin settings field |

Skills are defined in `.claude/skills/` — read the relevant `SKILL.md` for the full step-by-step procedure.

---

## Plugin Identity

- **Display name:** Consentric — Simple Cookie Consent
- **WP.org slug:** `consentric-cookie-consent`
- **Slug (files):** `consentric-cookie-consent`
- **Main file:** `consentric-cookie-consent.php`
- **GitHub:** https://github.com/sergiorsmaster/consentric
- **Min WP:** 6.0 | **Min PHP:** 7.4
- **Text domain:** `consentric-cookie-consent`

---

## Code Conventions

- PHP prefix `cscc_` on all functions, classes, hooks, and options.
- JS namespace `window.SimpleCookieConsent`.
- Wrap all user-facing strings with `__()` / `esc_html__()` / `esc_html_e()`.
- WordPress Settings API for all admin forms.
- Enqueue scripts/styles via `wp_enqueue_scripts` / `admin_enqueue_scripts`.
- No inline styles in PHP templates — use CSS files or `wp_add_inline_style`.
- Do NOT use `load_plugin_textdomain()` — WordPress.org handles translations automatically since WP 4.6.

### WordPress Security Rules (Plugin Check compliance)

These rules are **mandatory** — Plugin Check will flag violations as errors or warnings.

**Output escaping** — every `echo` must use an escaping function:
- Text: `esc_html()`, `esc_html_e()`, `esc_html__()`
- Attributes: `esc_attr()`, `esc_attr_e()`
- URLs: `esc_url()`
- HTML fragments built in PHP: `wp_kses_post()`
- Numbers in inline JS: cast with `(int)` or `intval()`

**Input handling** — always `wp_unslash()` before sanitizing superglobals:
- `sanitize_text_field( wp_unslash( $_POST['field'] ) )`
- `sanitize_key( wp_unslash( $_GET['key'] ) )`
- `absint( $_GET['id'] )` (absint is safe without unslash)
- Nonces: `wp_verify_nonce( sanitize_text_field( wp_unslash( $_POST['nonce'] ) ), 'action' )`

**Nonce verification** — required before processing any `$_GET` / `$_POST` data:
- For form submissions: use `wp_nonce_field()` + `wp_verify_nonce()`
- For read-only display params (e.g. `$_GET['tab']`): add `// phpcs:ignore WordPress.Security.NonceVerification.Recommended` with a comment explaining why

**Redirects** — always use `wp_safe_redirect()` instead of `wp_redirect()`.

**Direct database queries** — our custom table (`cscc_cookies`) requires `$wpdb` calls. Add a phpcs ignore comment:
```php
// phpcs:ignore WordPress.DB.DirectDatabaseQuery, WordPress.DB.PreparedSQL.InterpolatedNotPrepared -- $table uses trusted prefix
$results = $wpdb->get_results( "SELECT * FROM {$table} WHERE ..." );
```

**No inline JS event handlers** — use `data-cscc-action` attributes + delegated JS listeners instead of `onclick`.

---

## Repository Structure

```
consentric/
├── consentric-cookie-consent.php ← Main file (plugin header + bootstrap + CSCC_VERSION constant)
├── uninstall.php               ← Drops table + deletes all cscc_* options
├── readme.txt                  ← WordPress.org listing
├── .github/workflows/
│   └── release.yml             ← Auto-build zip + GitHub Release on vX.X.X tag push
├── .claude/skills/             ← AI slash commands (skills)
├── includes/
│   ├── class-cscc-activator.php     ← create_tables(), set_defaults(), seed_own_cookie()
│   ├── class-cscc-deactivator.php
│   ├── class-cscc-consent-store.php ← PHP cookie reader
│   ├── class-cscc-cookie-scanner.php
│   ├── class-cscc-wp-consent-api.php
│   ├── class-cscc-polylang.php
│   └── class-cscc-shortcodes.php    ← [cscc_cookie_list], [cscc_preferences]
├── admin/
│   ├── class-cscc-admin.php         ← Menu, tabs, register_settings(), handle_cookie_actions()
│   ├── views/
│   │   ├── tab-general.php         ← cscc_general settings group
│   │   ├── tab-appearance.php      ← cscc_appearance settings group
│   │   ├── tab-jurisdiction.php    ← cscc_jurisdiction settings group
│   │   ├── tab-integrations.php    ← cscc_integrations settings group + Debug
│   │   ├── tab-cookies.php         ← Cookie list + scanner (custom form, no Settings API)
│   │   └── tab-help.php            ← Read-only reference (CSS, shortcodes, JS API)
│   └── assets/
│       ├── admin.css
│       └── admin.js
├── public/
│   ├── class-cscc-public.php        ← enqueue_scripts(), render_banner(), render_modal()
│   ├── views/
│   │   ├── banner.php
│   │   ├── modal.php
│   │   └── preferences-icon.php    ← Cookie SVG icon
│   └── assets/
│       ├── cscc-banner.css
│       ├── cscc-banner.js           ← SimpleCookieConsent JS API + GTM integration
│       └── cscc-modal.js            ← Focus trap, aria, toggle state
└── languages/
    ├── consentric-cookie-consent.pot              ← Master template (source of truth)
    ├── consentric-cookie-consent-pt_PT.po/.mo
    ├── consentric-cookie-consent-pt_BR.po/.mo
    └── consentric-cookie-consent-de_DE.po/.mo
```

---

## Key Options Reference

| Option | Tab | Type | Default |
|--------|-----|------|---------|
| `cscc_enabled` | General | toggle | `'1'` |
| `cscc_banner_title` | General | text | `'We use cookies'` |
| `cscc_banner_text` | General | textarea | *(long default)* |
| `cscc_accept_label` | General | text | `'Accept All'` |
| `cscc_deny_label` | General | text | `'Deny All'` |
| `cscc_preferences_label` | General | text | `'Preferences'` |
| `cscc_modal_title` | General | text | `'Cookie Preferences'` |
| `cscc_modal_intro` | General | textarea | *(long default)* |
| `cscc_modal_save_label` | General | text | `'Save Preferences'` |
| `cscc_modal_deny_label` | General | text | `'Deny All'` |
| `cscc_privacy_policy_page` | General | page selector | `0` |
| `cscc_cookie_policy_page` | General | page selector | `0` |
| `cscc_imprint_page` | General | page selector | `0` |
| `cscc_show_preferences_icon` | General | toggle | `'1'` |
| `cscc_position` | Appearance | select | `'bottom-bar'` |
| `cscc_color_bg` | Appearance | color | `'#ffffff'` |
| `cscc_color_text` | Appearance | color | `'#111111'` |
| `cscc_color_accent` | Appearance | color | `'#0073aa'` |
| `cscc_logo_source` | Appearance | select | `'custom'` |
| `cscc_logo_url` | Appearance | URL | `''` |
| `cscc_border_radius` | Appearance | number | `'6'` |
| `cscc_banner_max_width` | Appearance | number | `'200'` |
| `cscc_banner_border_width` | Appearance | number | `'0'` |
| `cscc_banner_border_color` | Appearance | color | `'#dddddd'` |
| `cscc_button_style` | Appearance | select | `'outline'` |

| `cscc_jurisdiction` | Jurisdiction | select | `'gdpr'` |
| `cscc_ccpa_opt_out_text` | Jurisdiction | text | *(fallback in render)* |
| `cscc_gtm_enabled` | Integrations | toggle | `'0'` |
| `cscc_gtm_mode` | Integrations | select | `'basic'` |
| `cscc_gtm_wait_for_update` | Integrations | number | `'500'` |
| `cscc_debug` | Integrations | toggle | `'0'` |

---

## JavaScript API (`window.SimpleCookieConsent`)

| Method | Description |
|--------|-------------|
| `acceptAll()` | Accept all categories, update GTM, hide banner |
| `denyAll()` | Deny all non-necessary, update GTM, hide banner |
| `saveConsent(obj)` | Save partial consent `{analytics, marketing, functional}` |
| `openPreferences()` | Open the preferences modal |
| `openBanner()` | Re-show the consent banner |
| `hasConsent(category)` | Returns `true/false` for a category |
| `hasInteracted()` | Returns `true` if visitor has made a choice |

Event: `document.addEventListener('cscc:consentUpdated', e => { e.detail /* consent object */ })`

---

## Feature / Task List

Status: `[ ]` pending | `[~]` in progress | `[x]` done

### Phase 0 — Project setup
- [x] FEAT-01: Docker dev environment
- [x] FEAT-02: Plugin scaffold

### Phase 1 — Core consent engine
- [x] FEAT-03: Cookie consent storage (JS cookie + PHP reader)
- [x] FEAT-04: GTM Consent Mode v2 — default + update signals (Basic mode)
- [x] FEAT-05: GTM Advanced mode support

### Phase 2 — Banner UI
- [x] FEAT-06: Banner HTML template + CSS
- [x] FEAT-07: Accept All / Deny All / Preferences modal
- [x] FEAT-08: Re-open preferences via footer link

### Phase 3 — Admin settings
- [x] FEAT-09: Admin menu + General settings tab
- [x] FEAT-10: Appearance tab
- [x] FEAT-11: Jurisdiction tab
- [x] FEAT-12: Integrations tab

### Phase 4 — Cookie scanner
- [x] FEAT-13: DB table + admin cookie list UI
- [x] FEAT-14: Cookie scanner (PHP header scan + JS client scan)
- [x] FEAT-15: Open Cookie Database lookup + built-in fallback list

### Phase 5 — Integrations & shortcode
- [x] FEAT-16: WP Consent Level API integration
- [x] FEAT-17: Polylang compatibility
- [x] FEAT-18: [cscc_cookie_list] shortcode

### Phase 6 — Polish & release
- [x] FEAT-19: Uninstall cleanup
- [x] FEAT-20: readme.txt for WordPress.org
- [x] FEAT-21: Banner & modal style review + expanded appearance settings

### Phase 7 — Developer experience & repository health
- [x] FEAT-22: Register cscc_consent cookie in DB on activation
- [x] FEAT-23: Replace preferences icon with cookie SVG
- [x] FEAT-24: Banner preview via ?cscc_preview=1 + admin link
- [x] FEAT-25: Admin Help tab
- [x] FEAT-26: Accessibility review (focus trap, tabindex, aria-checked)
- [x] FEAT-27: README.md for developers
- [x] FEAT-28: GitHub Actions release workflow
- [x] FEAT-29: License fix (GPL v2) + CODE_OF_CONDUCT.md
- [x] FEAT-30: UX improvements — button order, modal text, openBanner()
- [x] FEAT-31: i18n — PT-PT, PT-BR, DE translations + POT file

### Backlog
- [ ] FEAT-32: Script Manager — admin UI to manage custom script snippets per consent category

---
> Source: [sergiorsmaster/consentric](https://github.com/sergiorsmaster/consentric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
