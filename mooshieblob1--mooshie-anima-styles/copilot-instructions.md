## mooshie-anima-styles

> A static single-page gallery for browsing artist style tags from the [Anima](https://civitai.com/models/118825/anima-pencil-xl) AI image generation model. Users search, browse, sort, favourite, and copy artist tags. All artist data (manifest, shards, search index, images) is served from a remote CDN — the app has **zero backend/API of its own**.

# AGENTS.md

## Project Overview

A static single-page gallery for browsing artist style tags from the [Anima](https://civitai.com/models/118825/anima-pencil-xl) AI image generation model. Users search, browse, sort, favourite, and copy artist tags. All artist data (manifest, shards, search index, images) is served from a remote CDN — the app has **zero backend/API of its own**.

## Commands

```bash
pnpm install      # install deps
pnpm dev          # vite dev server
pnpm build        # production build → dist/
pnpm preview      # preview production build
```

There are no test, lint, or format commands configured. No CI workflows exist.

### Utility Scripts

- `node scripts/gen-icons.mjs` — regenerates the favicons and the 600×600 square `og-image.png` social thumbnail in `public/` from `src/icons/icon.png` using sharp. Run manually after changing the source icon.

## Tech Stack

- **Svelte 5** with runes (`$state`, `$derived`, `$effect`, `$props`, `$bindable`) — not Svelte 4 syntax
- **TypeScript** (strict mode, `noEmit`)
- **Tailwind CSS v4** via `@tailwindcss/vite` plugin — imported as `@import "tailwindcss"` in `app.css`, no `tailwind.config` file
- **Vite 6** bundler
- **pnpm** package manager (workspace config disables native builds for esbuild/sharp/workerd)
- **Lenis** — smooth scroll library (only runtime dependency)

## Architecture

### Data Flow

```
CDN (cdn.mooshieblob.com)
  └─ manifest.json          → describes shards, image base URL, search index path
       ├─ search.json       → flat array of ArtistSearchHit (slug, tag, postCount, shard, imageId)
       └─ shards/{a..z,_}.json → full ArtistEntry records keyed by slug
```

1. `App.svelte` passes a hardcoded manifest URL to `ArtistGalleryPage`
2. `ArtistGalleryStore` (Svelte 5 class with `$state` fields) initialises by fetching the manifest, then eagerly loads the search index
3. The gallery page renders cards from the search index; full shard data is fetched on-demand when opening the lightbox (for aliases)
4. Images are served as `.webp` files at `{imageBaseUrl}/{releasePrefix}/images/{imageId}.webp`

### Module Structure

All gallery logic lives in `src/lib/artist-gallery/` as a self-contained, portable module:

| File | Role |
|---|---|
| `types.ts` | All TypeScript interfaces (mirrors Python build scripts' output shapes) |
| `client.ts` | `createArtistGalleryClient()` — fetch-based data layer with in-memory caching |
| `store.svelte.ts` | `ArtistGalleryStore` class — reactive state using Svelte 5 runes; singleton per manifest URL |
| `index.ts` | Barrel re-exports for the module |
| `components/ArtistGalleryPage.svelte` | Main page component (~1200 lines) — search, sort, pagination, favourites, categories, infinite scroll |
| `components/ArtistLightbox.svelte` | Full-screen image preview with zoom, keyboard nav, copy/insert actions |
| `components/ArtistCard.svelte` | Standalone card component (used for external integrations; the gallery page inlines its own card rendering) |
| `components/ArtistHoverPreview.svelte` | Tooltip-style preview for embedding in other apps |
| `components/ColorPicker.svelte` | Custom HSV colour picker for favourite categories |

### Key Patterns

- **Singleton stores**: `createArtistGalleryStore()` deduplicates by manifest URL using a module-level `Map`
- **Sequence-guarded search**: `setQuery()` uses an incrementing `searchSeq` counter to discard stale async results
- **Client caching**: manifest is cached for 60s; shards and search index are cached indefinitely until `invalidate()` is called
- **Tag normalisation**: strips leading `@`, lowercases, replaces spaces with underscores, strips backslashes
- **Shard bucketing**: first character of slug determines shard file (`a`–`z`, `0`–`9`, or `_` for special chars)

## Gotchas

- **`ArtistGalleryPage` is the real card renderer**, not `ArtistCard`. The page inlines card markup directly for performance (favourites, categories, copy-on-right-click, animations). `ArtistCard` exists as a simpler standalone component for external consumers.
- **Light theme is implemented by inverting Tailwind's neutral scale** via CSS custom properties in `app.css` (`.light` class on `<html>`). The codebase authors all styles for dark mode, and light mode flips the palette. Don't add separate light-mode utility classes.
- **No router** — it's a single page. The entire app mounts into `#app`.
- **`a11y_label_has_associated_control` warning is suppressed** in `svelte.config.js`.
- **Favourites and categories persist in `localStorage`** (`favourites`, `favCategories`, `theme`, `cardSliderVal`, `infiniteScroll` keys). Import/export uses a `{ version: 1, favourites, categories }` JSON shape.
- **Image preloading**: a sliding window of ±4 pages (or lookahead batch in infinite scroll mode) preloads images into a `Map<string, HTMLImageElement>` cache.
- **Uniqueness sort** uses a log-normal distribution (mu=5.5, sigma=1.8) with random jitter to surface "hidden gem" artists. The jitter regenerates each time the user clicks "Rotate".
- **Keyboard hotkeys**: keys `4`–`0` snap card size to preset densities; any printable character auto-focuses the search bar. Both are suppressed inside inputs/textareas.
- **Security headers** are in `public/_headers` (Cloudflare Pages format) — CSP restricts `img-src` and `connect-src` to `self` + `cdn.mooshieblob.com`.

## Deployment

Static site deployed to **Cloudflare Pages**. The `public/_headers` file provides security headers (CSP, X-Frame-Options, CORS restricted to `anima.mooshieblob.com`). The `.wrangler/` directory is present for local Cloudflare tooling.

## Style Conventions

- Tailwind utility classes for all styling; minimal custom CSS (only `app.css` overrides and a card slide-in animation in a `<style>` block)
- Svelte 5 runes exclusively — no `$:` reactive statements, no `export let`, no stores from `svelte/store`
- Component props defined via `interface Props` + `$props()` destructuring
- Event handlers use `on{event}` prop pattern (e.g., `onclose`, `oninsertTag`), not Svelte 4 `on:event` directives
- Functions prefixed with `display` for UI formatting (e.g., `displayTag`), `format` for number formatting (e.g., `formatCount`)

---
> Source: [Mooshieblob1/mooshie-anima-styles](https://github.com/Mooshieblob1/mooshie-anima-styles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
