## furuhashifumihito-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based bilingual (Japanese/English) academic research homepage for a University of Tokyo graduate student. Deploys to GitHub Pages (user site) at `https://furuhashifumihito.github.io/` via the GitHub Actions workflow in `.github/workflows/pages.yml`.

## Build Commands

```bash
# Install dependencies
bundle install

# Local development server (http://localhost:4000)
bundle exec jekyll serve

# Build static site to _site/
bundle exec jekyll build
```

## Architecture

### Jekyll Structure
- `_layouts/` - Template hierarchy:
  - `default.html` (base)
  - `home.html` extends `default`
  - `publications.html` (plural, list page) extends `default` — renders `site.data.bibliography`
  - `publication.html` (singular, detail page) extends `default` — rendered for each BibTeX entry at `/projects/<key>/`
- `_includes/nav.html` - Bilingual navigation component
- `_data/i18n.yml` - Bilingual labels keyed by `ja` / `en` (accessed via `site.data.i18n[page.lang]`)
- `_data/diary.yml` - Diary entries (daily memo style) with `date`, `text_ja`, `text_en` fields
- `publications.bib` - **Single source of truth for the publications list page** (BibTeX)
- `_plugins/bibtex_publications.rb` - Jekyll generator that parses `publications.bib` at build time, populates `site.data.bibliography`, and emits one virtual detail page per entry at `/projects/<key>/`
- `_publications/<bibtex_key>/` - Per-paper sidecar folder (one folder per paper) holding `meta.yml` (graphical abstract + figures), optional `body_ja.md` / `body_en.md` (Notes body), and image files. Not a Jekyll collection — read manually by the plugin.

### Hero Banner Slideshow

The top banner on the home layout (`_layouts/home.html`) is an auto-advancing
slideshow driven by files placed in `assets/images/hero/`. To add or reorder
slides, just edit the folder contents — no template or config changes needed.

- **Supported extensions**: `.jpg`, `.jpeg`, `.png`, `.webp`
- **Order**: alphabetical by filename. Use a numeric prefix
  (`01-`, `02-`, ...) to control it.
- **Single image**: rendered statically (the rotation JS is a no-op).
- **Multiple images**: crossfade every 5 s (1 s fade). Respects
  `prefers-reduced-motion`.
- **Performance**: the first slide gets `loading="eager"` +
  `fetchpriority="high"` for LCP; the rest are `loading="lazy"`.

The slideshow is assembled at build time by Liquid iterating over
`site.static_files` filtered to `/assets/images/hero/`. Styling lives in
`style.css` under `.hero__banner` / `.hero__slide`; the rotation script is
inlined near the bottom of `_layouts/home.html`.

### Bilingual System
- Japanese pages: `index.html`, `publications.html`, `diary.html`
- English pages: suffix `-e.html` (e.g., `index-e.html`, `publications-e.html`)
- Templates use `page.lang` variable for conditional content:
  ```liquid
  {% if page.lang == 'en' %}English{% else %}日本語{% endif %}
  ```

### Publications — Data Flow

The publications system is driven by a single BibTeX file plus an
optional per-paper sidecar folder. Everything is processed by one
generator (`_plugins/bibtex_publications.rb`).

**1. The list page (`publications.html` / `publications-e.html`)**

`publications.bib` is the single source of truth. At build time the
generator parses it, normalizes each entry, and populates
`site.data.bibliography`. `_layouts/publications.html` iterates over
that array. Adding or editing an entry on the list page is purely a
matter of editing `publications.bib`.

Each entry is normalized to:
```yaml
key:      "furuhashi2025dgpinn_itsc"    # BibTeX entry key
slug:     "furuhashi2025dgpinn_itsc"    # URL slug
title:    "Paper Title"
authors:  "Author Names"                # "Last, First and ..." → "First Last, ..."
venue:    "..."                         # journal / booktitle / howpublished / ...
year:     2025
type:     "conference"                  # journal | conference | domestic
links:    { paper: "...", pdf: "...", github: "...", ... }
abstract: "Abstract text"
bibtex:   "@inproceedings{...}"         # pretty-printed BibTeX block
lang:     "ja"                          # ja | en (from BibTeX `lang` field)
url:      "/projects/<slug>/"
```

Category resolution (`type` field):
1. If the BibTeX entry has a `bib_category = {journal|conference|domestic}`
   field, that value wins.
2. Otherwise, inferred from the BibTeX entry type:
   - `@article` → `journal`
   - `@inproceedings`, `@conference` → `conference`
   - `@misc`, `@techreport`, `@unpublished`, other → `domestic`

Entries are sorted by year descending at build time; within a year, the order
in the `.bib` file is preserved. The list page renders with `<ol reversed>`.

**2. Per-paper detail pages (auto-generated)**

Every BibTeX entry automatically gets a detail page at `/projects/<key>/`,
rendered by `_layouts/publication.html`. No hand-written page is needed —
the plugin creates a virtual `Jekyll::Page` per entry from the normalized
record above.

**3. Per-paper sidecar folder: `_publications/<bibtex_key>/`**

For papers that need extra content (graphical abstract, figures, notes,
images), create a folder named after the BibTeX key. One folder per
paper, everything co-located:

```
_publications/
  furuhashi2025dgpinn_itsc/
    meta.yml                # graphical_abstract + figures metadata
    body_ja.md              # (optional) Japanese "Notes" body
    body_en.md              # (optional) English "Notes" body
    graphical-abstract.png  # image files live in the same folder
    fig1-architecture.png
    fig2-loss.png
```

The generator reads this folder (if present), merges `meta.yml` into the
virtual page's front matter, injects the matching `body_<lang>.md` into
`page.content`, and copies every image into `/projects/<key>/<filename>`
via `Jekyll::StaticFile`.

`_publications/` is NOT a Jekyll collection — Jekyll ignores it because
of the leading underscore and the plugin handles everything manually.
Entries without a folder simply render as plain auto-generated detail
pages; the sidecar is purely additive.

**meta.yml schema:**

```yaml
graphical_abstract:
  src: graphical-abstract.png     # relative filename → /projects/<key>/graphical-abstract.png
  alt_ja: "日本語 alt"              # optional
  alt_en: "English alt"            # optional

figures:
  - src: fig1-architecture.png
    caption_ja: "図 1 のキャプション"
    caption_en: "Fig. 1 caption"
  - src: fig2-loss.png
    caption_ja: "..."
    caption_en: "..."
```

Path resolution for `src`:
- **Relative filename** (e.g., `graphical-abstract.png`) — auto-resolved
  to `/projects/<bibtex_key>/graphical-abstract.png` and the file is
  copied to that URL at build time.
- **Absolute path** (starts with `/`) — used as-is (e.g., referencing
  a shared asset under `/assets/...`).
- **External URL** (starts with `http://` / `https://`) — used as-is.

**Body files:**

- `body_ja.md` is injected into `page.content` when the virtual page's
  `lang == "ja"` (default).
- `body_en.md` when `lang == "en"` (set via `lang = {en}` in BibTeX).
- `body.md` acts as a language-neutral fallback.
- Markdown is converted to HTML via Jekyll's configured converter
  (kramdown) before injection. `_layouts/publication.html` wraps it in
  the "Notes" block.

**Rendering rules (see `_layouts/publication.html`):**

- `graphical_abstract` renders as a prominent block directly below the
  title, before authors/venue.
- `figures` renders as a labelled list inside the body column after the
  abstract, with auto-numbered captions (`図 N.` / `Fig. N.`) picked
  from `_data/i18n.yml`.
- Captions use `caption_ja` / `caption_en` by language; `caption` alone
  is a language-neutral fallback. `alt_ja` / `alt_en` behave the same.

### Adding a Publication

1. Append a BibTeX entry to `publications.bib`.
2. (Optional) If the paper needs a graphical abstract, figures, or notes,
   create `_publications/<bibtex_key>/` and drop in `meta.yml`, any
   `body_ja.md` / `body_en.md`, and the image files.
3. `bundle exec jekyll serve` — the generator logs how many entries it
   loaded; verify the paper appears in the expected section and that
   `/projects/<key>/` renders correctly.

### URL Generation
Always use Liquid filters for URLs:
```liquid
{{ '/index.html' | relative_url }}
{{ '/style.css' | relative_url }}
```

## Key Configuration

- `_config.yml`: Site settings, markdown converter, excludes
- `baseurl: ""` - User site is served at the domain root, so no subpath prefix
- No Jekyll collections are defined — per-paper content is assembled by
  `_plugins/bibtex_publications.rb` at build time.

---
> Source: [FuruhashiFumihito/furuhashifumihito.github.io](https://github.com/FuruhashiFumihito/furuhashifumihito.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
