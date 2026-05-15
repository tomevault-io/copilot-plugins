## photos

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Commands

```bash
npm install       # install dependencies
npm run dev       # dev server at http://localhost:5173
npm run check     # Svelte/TypeScript type checks
npm run build     # production build into build/
npm run preview   # preview production build

# Import photos from a source directory
npm run process-photos -- /path/to/LR_processed

# Delete a category
npm run process-photos -- --delete-category <category-name>

# Fill GPS for all photos in a category missing lat/lng (reverse-geocodes once)
npm run process-photos -- --fill-gps <category-name> --lat <lat> --lng <lng>
```

Minimum validation before any change: `npm run check && npm run build`.

## Architecture

**SvelteKit static site** using `@sveltejs/adapter-static`. All pages are prerendered. The production base path is `/photos` (set in `svelte.config.js` via `NODE_ENV`).

### Data flow

`src/lib/photo-data.json` is the single source of truth for all photo metadata. It is read at build time by `src/routes/+page.ts` (via `prerender = true`) and passed to Svelte components as page data. No runtime API calls.

### Routing

- `/` — Gallery homepage. Shows category cover cards, or a photo grid when `?category=<name>` is set.
- `/photo/[slug]` — Photo detail page with EXIF info, map (when GPS present), and prev/next navigation within the same category.
- `/about` — About page.

### Category system

Categories are derived from the `category` field in `photo-data.json` — no hardcoded list. Album cover: first photo with `isCategoryCover: true` wins; falls back to the most recent photo.

### Photo processing script (`scripts/process-photos.js`)

Node.js CLI using `sharp` (resize/convert) and `exifr` (EXIF + GPS extraction). On import it:
1. Scans source subfolders as categories
2. Skips already-imported photos (matched by `sourcePath`, or legacy `category + slug`)
3. Generates `<category>_01`, `<category>_02`, ... filenames
4. Writes WebP full images (max 2400px, quality 82) and thumbnails (800×800, quality 82) to `static/images/<category>/`
5. Extracts GPS via `exifr.gps()` and reverse-geocodes to a place name using Nominatim (rate-limited to 1 req/sec)
6. Appends new entries to `src/lib/photo-data.json`

The `--fill-gps` mode updates existing entries that are missing `lat`/`lng`, calling Nominatim once for the whole category.

### Map display

When a photo has `lat` and `lng`, the detail page renders an OpenStreetMap iframe and a "View larger map" link.

## Coding style

- Tabs for indentation; TypeScript throughout
- `Photo` and `CategoryCover` interfaces defined in `src/routes/+page.ts` and reused across routes
- Always use `{base}/` prefix from `$app/paths` for image `src` or `href` attributes (required for the `/photos` base path in production)
- No dedicated formatter/linter — match surrounding style
- PascalCase for interfaces, camelCase for variables/functions, kebab-case or `category_NN` for image filenames/slugs

## Commit style

Short, imperative messages: `Add GPS extraction`, `Fix category filter`. No placeholder commits.

---
> Source: [hydrotian/photos](https://github.com/hydrotian/photos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
