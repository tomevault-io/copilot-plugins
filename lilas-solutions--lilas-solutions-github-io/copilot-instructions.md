## lilas-solutions-github-io

> These apply to **all** requests that involve content or code changes:

# Lilas Solutions Website - AI Agent Instructions

## Key Principles

These apply to **all** requests that involve content or code changes:

1. **Bilingual content** — All content pages must exist in both English (`content/en/`) and Spanish (`content/es/`). Keep translations in sync.
2. **Hugo build must succeed** — Run `hugo` to verify the site builds without errors after changes.
3. **Spanish accent rules** — Spanish text must use proper diacritical marks (tildes, accents, `¿...?`, `¡...!`).

## Project Overview

Hugo static site serving as the marketing website for Lilas Solutions at `lilas.solutions`. Single-page scrolling design using the `hugo-scroll` theme (git submodule).

Products showcased: MiniBiz, Stars, Study-bot
Languages: Spanish (default), English
Deploy: GitHub Actions → GitHub Pages

## Structure

```
content/
  en/                          # English content
    _index.md                  # Homepage header
    homepage/                  # Sections: intro, products, services, contact
    minibiz.md, stars.md, studybot.md  # Product pages
    *-terms-service.md, *-privacy-policy.md, *-delete-account.md  # Legal
  es/                          # Spanish content (mirrors English)
layouts/
  _default/                    # Product page and document page templates
  shortcodes/                  # card, card_row, screenshot
  partials/                    # custom_head
static/images/                 # Product icons, localized screenshots
assets/
  images/                      # Logo, favicon, cover
  css/variables.scss           # SCSS style overrides
themes/                        # hugo-scroll (git submodule)
```

## Technology

- **Hugo** v0.128.0 extended
- **Theme**: `hugo-scroll` (single-page scrolling) — git submodule from `zjedi/hugo-scroll`
- **Styling**: Dart Sass + SCSS variables
- **Content**: Markdown with TOML front matter
- **Custom shortcodes**: `card`, `card_row`, `screenshot`

## Build & Deploy

```
hugo                    # Build site (output: public/)
hugo server             # Local dev server with hot reload
```

Deployment is automatic via GitHub Actions on push to `main`:
1. Install Hugo Extended + Dart Sass
2. Checkout with `submodules: recursive`
3. `hugo --gc --minify`
4. Deploy to GitHub Pages

Custom domain: `lilas.solutions` (via `CNAME` file)

## Configuration

- `hugo.toml` — Main config (languages, menus, theme params)
- `.gitmodules` — Theme submodule reference

## Adding Content

- **New product pages**: Create in both `content/en/` and `content/es/` using `product_page` layout
- **New legal pages**: Use `product_document_page` layout
- **Homepage sections**: Add to `content/{lang}/homepage/` with appropriate `weight` for ordering
- **Screenshots**: Place in `static/images/` with language-specific subdirectories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lilas-solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
