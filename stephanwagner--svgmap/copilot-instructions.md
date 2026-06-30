## svgmap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build       # compile src/ → dist/ (Rollup, all formats)
npm run watch       # rebuild on file changes
node test/assets.js # verify dist output files exist (run after build)
```

There is no automated test suite beyond the asset presence check in `test/assets.js`. The `prepublishOnly` script runs `npm run build && node test/assets.js`.

## Architecture

svgMap is a single-class JavaScript library (`src/js/core/svg-map.js`) that renders an interactive SVG world map with per-country data visualisation.

### Entry point and build

- **`src/js/index.js`** — re-exports the class and imports the SCSS entry point
- **`rollup.config.js`** — compiles to six formats in `dist/`:
  - `index.js` (ESM), `index.cjs` (CommonJS)
  - `svg-map.umd.js` / `svg-map.umd.min.js` (UMD, global name `svgMap`)
  - `svgMap.js` / `svgMap.min.js` — legacy UMD aliases kept for backwards compatibility with pre-2.18 users
  - `svg-map.css` / `svg-map.min.css` (extracted from SCSS); legacy `svgMap.css` copies created after each build

### Core class (`src/js/core/svg-map.js`)

The `svgMap` class is instantiated directly by consumers (`new svgMap({ targetElementID, data, ... })`). Constructor calls `init()` which:
1. Validates `targetElementID` and `data` options
2. Builds the DOM structure: `.svgMap-wrapper` → `.svgMap-container` → `.svgMap-map-container` → `.svgMap-map-wrapper` → `<svg viewBox="0 0 2000 1001">`
3. Creates zoom controls and optional continent selector
4. Calls `createMap()` then `applyData()` — `applyData` computes per-country colors using `calculateColorRatio` and writes them as the CSS custom property `--svg-map-country-fill` on each country `<path>` element
5. The floating tooltip is appended to `<body>` and positioned with `showTooltip(e)` / `moveTooltip(e)`
6. Persistent tooltips (pinned labels on the map) are handled separately in `createPersistentTooltips()`

Pan and zoom are delegated entirely to the `svg-pan-zoom` library (the only runtime dependency).

Country paths are inlined inside the class as SVG `d` strings, keyed by ISO 3166-1 alpha-2 code. The `countries` property holds the name map; `emojiFlags` holds emoji representations. `EH` (Western Sahara) can be merged with `MA` (Morocco) via `options.countries.EH = false`.

Color interpolation (`getColor`) works by linearly blending two hex colors. `calculateColorRatio` supports `'linear'`, `'log'`, or an arbitrary `(value, min, max) => ratio` function.

### Styling (`src/scss/`)

- `svg-map.scss` — entry point, `@use`s `map` and `tooltip` partials
- `variables.scss` — all `!default` SCSS variables; consumers can override by importing with their own values before the library SCSS
- Country fill is driven by the `--svg-map-country-fill` CSS custom property set at runtime, not by a static class

### Demos

| Path | Description |
|---|---|
| `demo/html/` | Plain-HTML CDN demo, no bundler |
| `demo/es6/` | ES module demo (requires `npm install` inside the folder) |
| `demo/react/` | Create React App demo (requires `npm install`) |

Localization files (country name translations) live in `demo/*/local/`.

### Developer assets (`assets/`)

- `create-map-paths.html` — browser tool: paste SVG `<path>` elements into the embedded `<svg>` to visualize them and generate country path data; used when adding or editing country shapes
- `localize.html` — browser tool for generating `countryNames` translation objects
- `countries.js` — country code → name map used by the above tools

These files are shipped in the npm package but are not part of the library bundle.

### Publishing

Run `npm publish` from the repo root — `prepublishOnly` rebuilds and validates assets automatically. The `assets/` and `src/` directories are included in the published package alongside `dist/`.

---
> Source: [StephanWagner/svgMap](https://github.com/StephanWagner/svgMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
