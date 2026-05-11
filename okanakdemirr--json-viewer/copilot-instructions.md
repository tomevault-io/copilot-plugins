## json-viewer

> Project notes for AI assistants working in this repo.

# CLAUDE.md

Project notes for AI assistants working in this repo.

## What this is

Single-page dev tool hosting two utilities under tabbed navigation:
- **JSON viewer** — paste/upload, tree view with lazy expansion, in-tree search, image detection with lightbox gallery
- **Text comparer** — two-pane diff backed by `diff.js` (vendored `diff-match-patch`, loaded as a global `Diff`)

Plus a shared history sidebar (last 10 per type, localStorage-backed).

## Layout

```
index.html              — single page, references app.js (bundled) and diff.js
app.js                  — GENERATED BUNDLE from src/. Do NOT edit directly. Rebuild with `npm run build`.
styles.css              — dark theme, CSS custom properties defined in :root
diff.js                 — vendored diff-match-patch (global Diff)
src/
  app.js                — entry: JSONViewer coordinator + DOM bindings
  utils.js              — pure helpers (getType, isUrl, isImageUrl, truncateUrl, getThumbnailUrl, span, getDataAtPath) + IMAGE_EXTENSIONS / IMAGE_PATTERNS
  treeRender.js         — DOM-building tree renderer; takes a ctx { jsonData, stats, imageData, showImages }
  search.js             — Search class (in-tree text search + highlight)
  gallery.js            — Gallery class (lightbox with focus trap)
  history.js            — HistoryPanel class + pure helpers previewFor, dedupeAndInsert
  compare.js            — Comparer class
  toast.js              — showToast(message, kind, duration)
  clipboard.js          — copyWithFeedback(text, button, opts)
  __tests__/            — Vitest unit tests (jsdom env)
```

## Dev workflow

```
npm install          # one-time
npm run build        # bundles src/app.js → app.js (IIFE, 46kb)
npm run dev          # watch mode for build
npm test             # vitest run (29 tests currently)
npm run test:watch   # watch mode
npm run lint         # eslint src
npm run format       # prettier --write on src
```

Open `index.html` directly — no server needed. The root `app.js` is the committed bundle; browsers load it as a classic script, so file:// works.

## Critical rules

- **Do not edit `app.js` at the root.** It is the esbuild output. Edit files under `src/` then `npm run build`. The pre-edit state of the bundle will be overwritten.
- **Never render user-controlled JSON via `innerHTML` string concatenation.** Tree rendering uses `createElement` + `textContent` + `setAttribute` inside a `DocumentFragment` specifically to eliminate XSS via escape-bypass. If you need to add a new tree node type, use the pattern in `src/treeRender.js`. `innerHTML` is only acceptable for trusted static markup (e.g., inline SVG icons with no interpolation).
- **Diff is a global from `diff.js`**, not a module import. The Comparer feature-detects it at init and disables the compare button if missing.
- **Do not reintroduce `alert()`**. Use `showToast()` from `src/toast.js` instead.
- **Icon SVGs inside buttons should carry `aria-hidden="true" focusable="false"`.** The button's `aria-label` carries the accessible name for icon-only controls.
- **Tree toggle buttons emit `aria-expanded`.** Any code that flips collapsed state must also flip `aria-expanded` on the corresponding `.json-toggle` button (see `_toggleNode`, `expandAll.finish`, `collapseAll`, `Search._updateHighlight`).

## Performance hot paths

- `expandAll` runs level-by-level via `requestAnimationFrame` and walks only newly inserted descendants for the next level — do not regress to a full `querySelectorAll` on the whole output between levels.
- `Search.clearHighlights` batches `normalize()` once per unique parent — do not call `normalize()` inside the per-mark loop.
- `collapseAll` bumps `this._expandGen` to cancel any in-flight expand rAF chain; preserve this if modifying either method.

## Testing tips

- Pure helpers live in `src/utils.js` and `src/history.js` (`previewFor`, `dedupeAndInsert`) — test these directly with Vitest. They run in jsdom.
- DOM-heavy classes (Search, Gallery, HistoryPanel, Comparer) don't have tests yet. If adding coverage, use `jsdom` (already the env) and provide fake DOM via `document.body.innerHTML`.

## Conventions

- 4-space indent, single quotes, semicolons, trailing commas es5, 120 col — enforced by Prettier config at `.prettierrc.json`.
- ES module syntax in `src/`. Class methods prefixed with `_` are module-internal.
- `document-at-path` expressions like `b.d[2].e` are the lazy-render path format; `getDataAtPath` is the canonical resolver.

## Gotchas

- `extractImageUrls` and array-level image preview rendering were removed (the call path was silently broken). If a future feature wants "show thumbnails above an array of image URLs", plan for a second pass since `imageData` is only populated as the string branch runs.
- The `gallery.getImageData` is a getter callback `() => this.imageData` — the JSONViewer swaps the imageData array on every `_renderJSON`, so Gallery must re-read it lazily, not cache.
- The compare tab's `Diff` global is loaded via a non-module `<script>` tag in `index.html` BEFORE the bundle. Changing script order breaks the feature-detect.

---
> Source: [okanakdemirr/json-viewer](https://github.com/okanakdemirr/json-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
