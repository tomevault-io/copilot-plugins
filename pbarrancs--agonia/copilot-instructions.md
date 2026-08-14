## agonia

> Brand presence website for **agonia**, an independent climbing apparel collective

# CLAUDE.md — agonia website

## What this project is

Brand presence website for **agonia**, an independent climbing apparel collective
based in Mexico. The site communicates the brand's identity, mission, and values
to the climbing and outdoor community. There is no e-commerce or purchasing flow —
the catalog is display-only.

## Latest Updates (2026-07-28)

- Inventory availability now comes from the workbook itself: missing values are normalized to `0`, and `Stock` is derived as `Unidades - Vendidas`.
- The catalog sync logic reads `inventario_agonia.xlsx` as the source of truth, so the site can use the Excel file directly to mark products as available or sold out.
- The old `fix_inventario.py` step is no longer required; its normalization logic now runs inside the existing sync scripts.
- Catalog card headings use UI-level display-name mapping (no manual edits to `productos.csv`).
- Catalog section titles use collection display names only: `Fall T`, `Cadenas`, `Back Logo`, `Lágrima`.
- Home page media section label is `Clips` (previously `Videos`).
- Home hero keeps only one CTA card (`Ver catálogo`); the extra `Nosotros` card was removed.
- Entry-point behavior reverted: `/` loads the home page directly (no Firebase redirect to `/catalogo`), and `/catalogo` remains available via navigation.
- Nosotros page now includes updated manifesto copy and a centered Perrito illustration block at the bottom.
- Current Perrito image source:
  `/assets/images/Ilustraciones/Perrito_03/P3_PNG_/P3_BlancoNegro_png/Perrito3_blanco_FondoOscuro.png`.

---

## Brand context

- **Category:** Independent climbing apparel (streetwear inspired by climbing culture)
- **Location:** Mexico
- **Audience:** Climbing and outdoor community in Mexico and Latin America
- **Tone:** Independent, aesthetic, community-driven
- **Social:** Instagram [@\_\_\_agonia](https://www.instagram.com/___agonia/) · YouTube [@agon_ia](https://www.youtube.com/@agon_ia)
- **Catalog rule:** Display only — no shopping cart, no checkout, no purchasing flow

---

## Current stack

| Tool                 | Version / Source                             | Role                                                     |
| -------------------- | -------------------------------------------- | -------------------------------------------------------- |
| **Astro**            | 4.16.x (`^4.16.0`)                           | Static site framework — builds to `new-astro-site/dist/` |
| **CSS**              | Plain CSS (no framework)                     | Layout, typography, brand colors                         |
| **Google Fonts**     | `New Rocker`, `Nova Cut`, `Syne Mono`        | Brand typography (loaded via CDN)                        |
| **Font Awesome 5**   | Bundled locally in `public/assets/webfonts/` | Social and UI icons                                      |
| **Vanilla JS**       | Inline `<script>` blocks in Astro components | Catalog interactivity (color/size filtering)             |
| **Firebase Hosting** | Project:`agonia-255fe`                       | Static site deployment and CDN                           |

No React, no Vue, no Svelte. No Tailwind, no PostCSS. No jQuery.

---

## Brand design tokens

```css
/* Defined in new-astro-site/src/styles/global.css */
--color-titulo: #e8431e; /* New Rocker — headings */
--color-subtitulo: #f4ec62; /* Nova Cut — subheadings */
--color-cuerpo: #fcfcfc; /* Syne Mono — body text */
--color-bg: #1a1a1a; /* Dark background */
```

---

## Current project structure

```
agonia/                                   ← repo root
├── firebase.json                         ← Firebase Hosting config; serves from new-astro-site/dist/
├── .firebaserc                           ← Firebase project alias: agonia-255fe (never modify)
├── .gitignore
├── README.md
├── CLAUDE.md                             ← this file
├── new-astro-site/                       ← Astro project (the live site)
│   ├── package.json                      ← astro ^4.16.0; scripts: dev / build / preview
│   ├── astro.config.mjs                  ← output: 'static'
│   ├── dist/                             ← build output; what Firebase deploys (gitignored)
│   ├── public/                           ← static assets copied as-is into dist/
│   │   └── assets/
│   │       ├── a_icono.ico               ← browser favicon
│   │       ├── css/
│   │       │   └── fontawesome-all.min.css
│   │       ├── images/
│   │       │   ├── bg.jpg                ← page background
│   │       │   ├── avatar.jpg / avatar_2.jpg
│   │       │   ├── nico_razo.gif / .mp4
│   │       │   ├── Fondos/               ← 7 background color variants
│   │       │   ├── fulls/ / thumbs/      ← gallery images (06 each)
│   │       │   ├── Logos/                ← Logo_A_lagrima, Logo_figura, Logo_horizontal (8 variants each)
│   │       │   ├── Ilustraciones/        ← Perrito_01–08, Personaje_A
│   │       │   └── playeras/             ← 29 product photos (PNG) + placeholder.svg
│   │       └── webfonts/                 ← 15 Font Awesome font files
│   └── src/
│       ├── env.d.ts
│       ├── data/
│       │   └── productos.csv             ← 144 rows; source of truth for catalog
│       ├── layouts/
│       │   └── Layout.astro              ← base HTML shell; loads global.css + Google Fonts
│       ├── components/
│       │   ├── Nav.astro                 ← site navigation with active-tab logic
│       │   ├── Footer.astro              ← footer with Aviso de Privacidad link
│       │   └── ProductCard.astro         ← catalog card: color swatches, dynamic size badges, OOS
│       ├── pages/
│       │   ├── index.astro               ← homepage — hero tagline "marca independiente"
│       │   ├── nosotros.astro            ← manifesto content + bottom Perrito illustration
│       │   ├── catalogo.astro            ← CSV-driven catalog with UI display-name mapping
│       │   └── aviso-de-privacidad.astro ← legal notice (Pablo Barranco Soto, Mayo 2026)
│       └── styles/
│           ├── global.css                ← brand tokens, typography, base layout
│           └── catalog.css               ← grid, card, swatch, size-badge styles
├── public/                               ← OLD static site — kept as backup until 2026-06-02
│   └── index.html                        ← do not delete before 2026-06-02
├── scripts/
│   ├── create_excel.py                   ← rebuilds inventario_agonia.xlsx from the markdown inventory table; safe by default and does not overwrite the workbook unless `--force` is passed
│   ├── process_ventas.py                 ← compatibility stub; inventory updates now happen directly in the workbook
│   ├── sync_oos.py                       ← syncs inventory → productos.csv disponible field
│   ├── convert_mod.py                    ← video conversion (not web-related)
│   └── convert_mts.py                    ← video conversion (not web-related)
├── ventas.csv                            ← legacy sales log; deprecated in the active workflow
├── inventario_agonia.xlsx                ← source of truth for stock; update this file directly when a sale happens
└── Documentos/                           ← brand docs, drafts, inventory — not web-related
```

---

## Firebase deployment

- **Config file:** `firebase.json` — serves from `new-astro-site/dist/`
- **Project alias:** `.firebaserc` → `agonia-255fe`
- **Build command:** `npm run build` (from `new-astro-site/`)
- **Deploy command:** `firebase deploy` (from repo root, after `firebase login`)
- **Live URL:** `https://agonia-255fe.web.app`

---

## Data pipeline

`productos.csv` is the canonical product source. Two Python scripts maintain it:

1. **`scripts/process_ventas.py`** — retained only as a compatibility stub; the workflow now
   updates inventory directly in `inventario_agonia.xlsx`.
2. **`scripts/sync_oos.py`** — normalizes `inventario_agonia.xlsx` (fills missing values with
   `0`, computes `Stock = Unidades - Vendidas`) and updates `disponible` in `productos.csv`.
   It also uses `CSV_COLOR_ALIAS = {'Morada': 'Morado'}` to bridge the color name difference
   between the CSV (`Morada`) and the Excel file (`Morado`).
3. **`scripts/create_excel.py`** — rebuilds `inventario_agonia.xlsx` from the markdown inventory
   table in `Documentos/playeras/detalles_agonia_playeras.md`. It is a backup/helper utility for
   reconstructing the workbook from the documented inventory source, not a regular inventory-editing
   tool. By default it does not overwrite an existing workbook; use
   `python scripts/create_excel.py --force` only when you intentionally want to replace it.

> Disclaimer: do not use this script as part of the normal sales workflow. It is not meant to
> edit inventory counts day to day, and it should not be run unless you explicitly want to
> reconstruct or reset the workbook from the markdown source.
> Run `scripts/sync_oos.py` after any inventory change, then rebuild and deploy.

---

## Migration plan — Phase checklist

- [x] **Phase 1 — Setup:** Created `new-astro-site/` with `package.json` and
      `astro.config.mjs` (output: static).
- [x] **Phase 2 — Content migration:** Built Layout, Nav, Footer components and styles.
      Migrated all existing HTML content into Astro pages (nosotros.astro, index.astro).
- [x] **Phase 3 — New features:** Catalog page (CSV-driven), aviso de privacidad page,
      active nav tab logic, footer link, per-color dynamic size filtering in ProductCard.
- [x] **Phase 4 — Assets:** All images and Font Awesome files copied into
      `new-astro-site/public/assets/`. All src paths updated in components.
- [x] **Phase 5 — Swap** ✅ **(completed 2026-05-26):**
      Updated `firebase.json` to point to `new-astro-site/dist/`. Deployed to Firebase.
      Live at https://agonia-255fe.web.app. `public/index.html` retained as backup
      (do not delete before 2026-06-02).
      Verified `.firebaserc` still references `agonia-255fe`.

---

## Development rules

### Files that must not be modified

- `public/index.html` — backup only; do not touch
- `.firebaserc` — never modify
- `new-astro-site/src/data/productos.csv` — updated by the sync scripts; do not hand-edit
- `inventario_agonia.xlsx` — source of truth for availability; update this file directly when a sale happens

### Technical constraints

- **Framework:** Astro only — no React, no Vue, no Svelte
- **CSS:** Plain CSS only — no Tailwind, no CSS-in-JS, no PostCSS plugins
- **No e-commerce:** No shopping cart, no checkout, no payment integration
- **Image references:** All images served from `/assets/images/` (inside `new-astro-site/public/assets/`)
- **Content:** Do not invent text — read from actual files and data sources only
- **CSV parsing:** `import rawCsv from '../data/productos.csv?raw'` + manual split in
  Astro frontmatter at build time — no external CSV parsing npm packages
- **Catalog prices:** Product prices are configured in `new-astro-site/src/pages/catalogo.astro`
  inside the `getPrecio()` lookup, not in `productos.csv`.

### Naming conventions observed in the project

- Image filenames use mixed case and underscores (e.g., `Perrito01_FondoClaro.png`)
- Known typos in existing filenames — copy as-is, do not rename:
  - `Perrito4_blanco_FondoOscuropng.png` (double extension)
  - `lgoo_naranja.png` (missing letter)
  - `gria_01.png` and `narajna_01.png` (spelling errors in Fondos)

### CSV color values (productos.csv)

`Morada`, `Negro`, `Blanco`, `Gris`, `Carbon`, `Navy`, `Rosa`, `Aqua`, `Verde`

Note: the Excel inventory uses `Morado` (not `Morada`). `sync_oos.py` bridges this with an alias map.

---

## Legacy

### Old stack (pre-migration, used until 2026-05-26)

| Tool                  | Role                                                                               |
| --------------------- | ---------------------------------------------------------------------------------- |
| **HTML5**             | Single-page static markup (`public/index.html`)                                    |
| **CSS / Sass (SCSS)** | Strata by HTML5UP (CCA 3.0) + custom brand styles; source in `public/assets/sass/` |
| **Google Fonts**      | Brand typography (same fonts as current)                                           |
| **Font Awesome 5**    | Bundled locally in `public/assets/webfonts/`                                       |
| **jQuery + Poptrox**  | DOM utilities and lightbox, inherited from the HTML5UP Strata base template        |
| **Firebase Hosting**  | Same project (`agonia-255fe`); serving from `public/`                              |

### Old file structure (web files)

```
public/
├── index.html              ← single-page site (all sections in one HTML file)
├── a_icono.ico
└── assets/
    ├── css/
    │   ├── main.css        ← compiled from Sass (do not edit directly)
    │   └── fontawesome-all.min.css
    ├── sass/
    │   ├── main.scss       ← Sass entry point
    │   └── libs/           ← _vars, _functions, _mixins, _vendor, _breakpoints, _html-grid
    ├── js/
    │   ├── main.js         ← parallax + touch handling (Strata)
    │   ├── jquery.min.js
    │   ├── jquery.poptrox.min.js
    │   ├── browser.min.js
    │   ├── breakpoints.min.js
    │   └── util.js
    └── webfonts/           ← Font Awesome font files
```

### Why the migration was done

- The old site was a single `index.html` with no multi-page support, no catalog,
  and no build-time data integration.
- The Astro migration enabled: a CSV-driven product catalog, per-page routing,
  reusable components, and a maintainable codebase — all while keeping the output
  fully static and compatible with Firebase Hosting.

### Date of migration

**2026-05-26** — `firebase.json` updated, Astro build deployed, site verified live.

---

## Current status

- [x] **Inventory update flow:** the workbook is now the single source of truth for stock and availability. When a sale happens, update the relevant row directly in `inventario_agonia.xlsx` and run `scripts/sync_oos.py` to refresh `productos.csv`.
- [x] **Legacy sales-table workflow:** the active inventory process no longer depends on `ventas.csv`.

---
> Source: [pbarrancs/agonia](https://github.com/pbarrancs/agonia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
