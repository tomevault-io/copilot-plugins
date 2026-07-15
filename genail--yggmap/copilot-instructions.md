## yggmap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**yggmap** (live at https://yggmap.com; storage keys deliberately keep the older `mindmaps.` prefix — renaming them would orphan users' data): a free, fully private, one-page mind-mapping web app in the Excalidraw spirit — canvas-is-the-app, local-first, keyboard-first, zero onboarding.

## Commands

There is **no build system or linter — on purpose**. Plain HTML/CSS/JS; the
`package.json` npm tooling exists only for the test suite and never ships.

```sh
python3 -m http.server 8471     # serve for development; file:// won't work (localStorage)
# → http://localhost:8471/index.html

npm test                        # headless Playwright suite in tests/ (one-time
                                # setup: npm install && npx playwright install chromium)

npx wrangler deploy             # deploy to production (yggmap.com, Cloudflare Workers
                                # static assets; one-time auth: npx wrangler login)
```

Deployment serves the repo root as-is; `.assetsignore` lists what never ships
(mockups, `assets/src` logo sources, docs, test tooling) — extend it when
adding dev-only files. Deploying is a production action: **only deploy when
the user asks.**

### Tests (`tests/`)

`npm test` drives the real app headless via `@playwright/test` (config in
`playwright.config.js`; it starts the http server itself, or reuses one already
running on 8471). Fixture maps are planted straight into `localStorage` by
`tests/helpers.js#seedMap` before page scripts run — the seed is idempotent
across reloads so persistence tests can assert the app's own saves. Each test
gets a fresh browser context, so storage starts clean for free. **A bug fix or
behavior change should come with a test here** (red/green when practical).
CI runs the suite on every push and PR (`.github/workflows/tests.yml`).

Exploratory verification still happens by hand in a browser (Playwright MCP). Practical notes for both:
- Bust caching by appending `?v=<anything>` to the URL after editing files.
- `window.__mm()` returns `{focusId, editingId, rootId, currentMapId, undoDepth}` for assertions.
- Glide/pan animations use `requestAnimationFrame`; in a **background** tab they crawl due to rAF throttling — a test artifact, not a bug. Time-based easing still converges.
- A live page re-saves state on unload (blur → commit → save), so wiping `localStorage` must happen before navigation, not on the page under test.
- In-app **debug mode**: `⌘⇧D`/`Ctrl+Shift+D`, persisted in the UI storage key. Currently reveals the 📱 mobile-preview button that forces the touch UI on desktop.

## Architecture

Four files, no modules, load order matters (`sketch.js` then `app.js`, both global-scope):

- `index.html` — app shell + shell-only CSS (positioning of fixed chrome, drawer/panel visibility). Component styling does NOT belong here.
- `styles.css` — the living design system: all `.mm-*` components and tokens. Shared by the app, `book.html`, and some mockups. **Any new/changed component must also be represented in `book.html`** (the component book is documentation-by-rendering).
- `sketch.js` — `MMSketch`: seeded-PRNG hand-drawn SVG connectors (jittered bezier + faint offset "pencil echo"). Seed comes from node-id hashes so lines don't rewiggle every render.
- `app.js` — everything else, organized in commented sections.

### Render pipeline (app.js)

`refresh()` runs the whole pipeline on every mutation:

```
doc (JSON tree) → buildShadow()   clone of visible tree with ghost entries injected
               → layoutShadow()   DOM-measure sizes → subtree-height stacking (H_GAP/V_GAP)
               → renderShadow()   reconcile DOM nodes in #nodes (className composition)
               → applyPositions() absolute left/top; lastPos = world-coord centers
               → drawLinks()      SVG connectors in #links
               → renderChrome()   body classes, hint strip, mobile action bar
```

Key ideas that span multiple functions:

- **Ghosts are real layout citizens.** `ghost:child` / `ghost:sibling` placeholders show what Tab/Enter would create and the map genuinely makes room for them. On desktop they're educational hints that fade with use (`learnLevel` capped at 2 — faint but **never fully hidden**; silent disappearance reads as a bug). On mobile they ARE the create buttons. Only real Tab/Enter keypresses increment learn counters — never taps.
- **Sticky sides.** Depth-1 branches persist `node.side` ('R'/'L'); new branches go to `lighterSide()`, siblings inherit. Prevents whole-branch reshuffling when the layout changes.
- **Viewport is a transform** on `#world` (`view = {x, y, s}`). `ensureVisible(id)` pans minimally (smooth `glideTo`) so the focused node *plus its ghosts* fit inside margins that account for chrome and the mobile keyboard (`visualViewport`); it runs on focus change, edit start, **and commit** (ghosts only reappear after editing ends). User gestures cancel glides.
- **Editing is in-place contenteditable.** Commit paths: Enter/Esc/Tab in `onEditKeydown`, blur in `onEditBlur`, and `setFocus` commits before moving. An empty *new* node is removed on commit (`removeQuiet`).
- **Undo/redo = whole-doc JSON snapshots** (`pushUndo`, max 100). Any new mutation must call `pushUndo()` first and `refresh()` after.
- **Persistence**: per-map docs under `mindmaps.map.<id>`, plus index/current/ui keys (schema table in README). Docs carry a `v` field (`DOC_VERSION`); **every doc load goes through `hydrateDoc()`** — that is where a future migration ladder hooks in. Import fails fast on payloads with a newer version. `saveDoc()` refuses to write when the map was just deleted (don't resurrect).
- **Multi-tab safety & sync**: the index is shared mutable state across tabs — every index mutation calls `reloadIndex()` first (read-modify-write, never write a stale copy; a microsecond interleave remains as accepted residual risk), and `saveDoc()` skips no-op writes (`lastSavedDoc`) so an idle tab can't overwrite fresh data with stale. A `storage` listener adopts remote changes live: index changes re-render the drawer / switch away from a remotely-deleted map; writes to the open map are adopted via `adoptRemoteDoc()`, which first pushes a pre-adoption undo snapshot (⌘Z = "before the remote write", never a time warp past it). Conflict policy is **last-writer-wins on whole docs**; a remote write arriving mid-edit is parked and adopted only if the commit changed nothing (and a placeholder the commit cancelled is re-dropped from the adopted doc). A remote doc with a newer `v` latches `docReadOnly` instead of being adopted or overwritten. Deleting the last map writes the successor in the same index update — never publish an empty index. Node ids carry per-tab entropy (`TAB_TAG`). Tests in `tests/multitab.spec.js` (two pages, one context).

### Hard-won invariants (do not "simplify" away)

- `pointerdown` on nodes/ghosts calls `preventDefault()`: iOS fires a synthesized mouse/focus pass *after* `pointerup`, which would blur the editor a ghost tap just opened and delete the new node.
- A blur within 400ms of `startEdit` refocuses instead of committing (tap-settling / keyboard-opening noise; a human can't intentionally leave that fast).
- Focused-node emphasis uses a 0-spread `box-shadow` ring, not `border-width` — a thicker border shrinks the content box and re-wraps the text.
- Empty contenteditable has no line box; node min-heights use `calc(1lh + …)` per variant.
- Learn-fade caps at level 2; ghosts never vanish entirely.

## Conventions

- Design language: hand-drawn/sketchy — Patrick Hand + Caveat, paper/card tokens, wobble border-radii (`--mm-wobble-*`), slight rotations. New UI must speak it; generic-flat elements stick out immediately.
- All component classes are `.mm-*` prefixed and live in `styles.css`; positioning of app chrome lives in `index.html`.
- `mockups/` is design history — don't extend it; new visual explorations get their own mockup file there.
- **Changelog**: the "what's new" dialog (`#changeslist` in `index.html`) holds dated entries; the newest entry's `data-date` drives the unseen dot on the trigger (compared against `ui.changesSeen`). After completing a **user-visible** change, ask whether it should get a changelog entry — and include the proposed entry in the question (real current date via `date`, bullets in the app's lowercase voice). Internal/dev-only changes don't qualify. New entries go on top; pruning old ones from the bottom is always safe.
- Commit messages: no Claude attribution or trailers (this is not an ISO-scope repo).

---
> Source: [genail/yggmap](https://github.com/genail/yggmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
