## wp-lonestar

> - This repository is the reusable `lonestar` parent framework. Keep client behavior in a child theme.

# Lonestar Parent Theme Agent Guide

## Scope

- This repository is the reusable `lonestar` parent framework. Keep client behavior in a child theme.
- Do not edit WordPress core.
- This LocalWP development checkout uses folder/stylesheet `wp-lonestar`; release archives keep the public ZIP root `lonestar/`.

## Runtime architecture

- `functions.php` loads core entrypoints in an explicit dependency order.
- Blocks are discovered from parent, active child, and enabled folder modules. Child block/module identities override parent identities.
- Content types are declarative direct PHP files in `inc/content-types/` from parent then active child; child entities override parent by slug, and at `init` priority 5 all post types register before taxonomies.
- `lonestar_get_content_type_catalog()` is the request-local source of truth for content-type runtime registration and the read-only Theme Settings `Content Types` overview; preserve the compatibility definitions API and do not add persistent catalog diagnostics or caches.
- Supported block roots are `blocks/acf`, `blocks/native`, and `blocks/php-only`.
- Native blocks are static (JavaScript `save`) or dynamic (`render.php`). PHP-only blocks require WordPress 7, `supports.autoRegister`, a `file:` render target, and no `editorScript`.
- ACF blocks/modules remain visible as unavailable when ACF Pro is absent; non-ACF runtime must keep working.
- Module `requires` and `admin_links` are explicit JSON metadata. Frontend state reads must not write options.
- `dist/` is tracked production output and ships in releases.
- Parent updates are disabled automatically when the installed parent contains `.git`; `LONESTAR_ALLOW_UPDATES` is the explicit override.
- Parent source strings use the sole `lonestar` text domain and English sources. Regenerate `languages/lonestar.pot` with `npm run i18n:pot` after changing translatable PHP, JavaScript, block metadata, or parent module metadata.

## CSS architecture

- Author standards-based CSS nesting only; `postcss-nesting` runs before Autoprefixer.
- Browser targets come from `@wordpress/browserslist-config` in `package.json`. Use `npm run browsers` to inspect the resolved list.
- Vite owns CSS `@import` handling. Do not add `postcss-import` or the Sass-like `postcss-nested` package.
- Tailwind is not part of this framework. Do not add Tailwind packages, configuration, directives, or generated utilities to parent or child themes.
- Keep `postcss`, `postcss-nesting`, `autoprefixer`, `browserslist`, and `@wordpress/browserslist-config` together unless the CSS pipeline contract is intentionally redesigned and documented.

## Build and validation

- Requirements: WordPress 7.0+, PHP 8.2+, Node.js 22.12+, npm 10+.
- Run from this directory:

    ```bash
    npm ci
    npm run check
    php tests/content-types-runtime.php
    php tests/content-types-admin.php
    find . -path './node_modules' -prune -o -type f -name '*.php' -print0 \
      | while IFS= read -r -d '' file; do php -l "$file" || exit 1; done
    git diff --check
    ```

- CI covers Node 22/24 contract/build/audit checks, the CSS pipeline contract, and PHP 8.2/8.4 lint. There is no PHPUnit or browser test suite.
- Runtime changes require a local frontend/admin/editor smoke test.

## Change discipline

- Prefix new public PHP symbols with `lonestar_`/`LONESTAR_`; preserve documented compatibility aliases.
- Use explicit module/block metadata, deterministic discovery, WordPress escaping and capability/nonce checks.
- Update `CHANGELOG.md`, relevant `docs/`, and this file when contracts or commands change.
- Normal feature work does not bump versions. Release prep aligns `style.css`, `package.json`, and `package-lock.json`.
- Releases are manual `workflow_dispatch` runs from `main`, with tag `lonestar-vX.Y.Z`, asset `lonestar-X.Y.Z.zip`, and ZIP root `lonestar/`.
- Do not commit, push, tag, publish, or activate/deactivate themes unless explicitly requested.

---
> Source: [blonestar/wp-lonestar](https://github.com/blonestar/wp-lonestar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
