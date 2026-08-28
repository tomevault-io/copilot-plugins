## hallowed-grounds-battlefield-tour-wp-theme

> A WooCommerce-ready [Roots Sage 11](https://roots.io/sage/) theme (Blade + Tailwind CSS v4 + Vite 8, powered by Acorn) for a Gettysburg, PA licensed-guide tour company concept. The visual design is ported from the static concept in [`matthummel-pa/tour-hallowed-ground-tours-theme`](https://github.com/matthummel-pa/tour-hallowed-ground-tours-theme).

# Hallowed Ground Battlefield Tours — WordPress Theme

A WooCommerce-ready [Roots Sage 11](https://roots.io/sage/) theme (Blade + Tailwind CSS v4 + Vite 8, powered by Acorn) for a Gettysburg, PA licensed-guide tour company concept. The visual design is ported from the static concept in [`matthummel-pa/tour-hallowed-ground-tours-theme`](https://github.com/matthummel-pa/tour-hallowed-ground-tours-theme).

## Cursor Cloud specific instructions

### What this repo is
This repository is only the **theme** (`wp-content/themes/hallowed-ground`). WordPress core is not part of the repo — a throwaway local WordPress install is created outside the repo to run/test the theme.

### System prerequisites (installed during initial VM setup)
PHP 8.3+ (with `mbstring xml curl zip gd intl sqlite3 bcmath`), Composer 2, WP-CLI (`wp`), and Node 20+/22+. The startup update script only refreshes the theme's own dependencies (`composer install`, `npm install`); it intentionally does **not** install system packages, build assets, or start services.

### First-time run in a fresh clone
Run the bootstrap helper — it is idempotent and safe to re-run:

```bash
bin/setup-wp.sh
```

This installs theme deps + builds assets, then stands up WordPress at `~/wp` using the **SQLite Database Integration** plugin (no MySQL needed), symlinks the theme in, installs + activates WooCommerce, seeds the three tour products, and activates the theme. Admin: `/wp-admin` (user `admin`, pass `admin123`).

### Running the site
```bash
wp server --path="$HOME/wp" --host=0.0.0.0 --port=8080 --allow-root
```
Then browse `http://localhost:8080/` (homepage) and `http://localhost:8080/shop/` (WooCommerce tours).

### Build / lint / dev
- Build assets (required — Vite generates the manifest the theme loads): `npm run build`
- Hot-reload dev server for assets: `npm run dev` (in addition to `wp server`)
- PHP lint (Laravel Pint): `./vendor/bin/pint --test` (drop `--test` to auto-fix)

### Non-obvious gotchas
- **You must build assets** (`npm run build`) or every page dies with a "Vite manifest not found" error. Build artifacts live in `public/build/` and are git-ignored.
- **Blade template changes are cached by Acorn.** After editing `.blade.php` files, if you don't see changes, clear the cache: `wp acorn view:clear --path="$HOME/wp" --allow-root` (or `wp acorn optimize:clear`).
- **Vite `base` path is hard-coded** in `vite.config.js` to `/wp-content/themes/hallowed-ground/public/build/`. If the theme is installed under a different folder name, update it or built asset URLs will 404.
- WooCommerce integration lives in `app/setup.php` (theme support + wrapper removal) and `resources/views/woocommerce.blade.php` (the Blade wrapper that renders shop/product/cart/checkout inside the theme layout). Booking CTAs point at the WooCommerce shop page.
- The site's content pages are **slug-based Blade templates**: the homepage is `resources/views/front-page.blade.php`, and `page-tours`, `page-guides`, `page-area`, `page-contact` render WordPress Pages with matching slugs (created by `bin/setup-wp.sh`). Editing `page-{slug}.blade.php` only takes effect if a published Page with that slug exists. Shared sections are in `resources/views/partials/` (`info-strip`, `book-band`).
- A harmless `_load_textdomain_just_in_time` notice is emitted by WooCommerce under WP 6.7+; it is not an error.
- **WooCommerce "Coming soon" mode** (Launch Your Store) defaults to ON and hijacks shop/product pages with a "Great things are on the horizon" placeholder that bypasses the theme. `bin/setup-wp.sh` disables it (`wp option update woocommerce_coming_soon no`); if the shop ever renders unstyled/placeholder, check that option first.

### Packaging & deploying the theme
- This is a WordPress theme — it ships as an installable theme package to a WordPress host.
- Build the installable zip with `bin/build-theme-zip.sh` → `dist-theme/hallowed-ground.zip`. The script ships the compiled `public/build` assets and a production (`--no-dev`) `vendor/` so the target host needs no composer/npm. Install via WP Admin → Appearance → Themes → Add New → Upload Theme, or `wp theme install dist-theme/hallowed-ground.zip --activate`.

---
> Source: [matthummel-pa/hallowed-grounds-battlefield-tour-wp-theme](https://github.com/matthummel-pa/hallowed-grounds-battlefield-tour-wp-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
