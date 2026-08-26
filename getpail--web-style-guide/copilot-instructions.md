## web-style-guide

> Static web style guide for PAIL web apps, built on Bootstrap 5.3.

# AGENTS.md

Static web style guide for PAIL web apps, built on Bootstrap 5.3.
No framework, no tests, no CI. Asset references are relative, so the site is
viewable by opening `index.html` in a browser. Default branch is `master`.

## Commands (Node >= 20 required)

- `npm run css` — compile `scss/bootstrapail.scss` -> `style/bootstrapail.css`
  (expanded, with source map), then minify -> `style/bootstrapail.min.css`.
  Individual steps: `npm run css-compile`, `npm run css-minify`.
- `npm run js` — copy Bootstrap's bundle from `node_modules` into
  `js/bootstrapail.bundle.min.js` (+ `.map`). Run after `npm install` bumps
  Bootstrap (`~5.3.8` pinned).

## Commit the generated files (important)

`style/*.css`, `style/*.css.map` and `js/bootstrapail.bundle.min.js(.map)`
are committed to git and linked directly from `index.html`. After any change
to `scss/`, run `npm run css` (and `npm run js` after a Bootstrap bump) and
commit the regenerated files. Never hand-edit anything under `style/` or
`js/bootstrapail.bundle.min.js` — they are overwritten. The sass step uses
`--no-error-css`, so on a compile error no file is written and the command
fails: check the exit status, don't trust a stale `style/` file.

## Architecture

- `index.html` — the guide's home page. `icons/index.html` — gallery of every
  icon in `images/sprites.svg`; keep its card list in sync when adding icons.
  `components/index.html` — demos of Bootstrap components using the guide's
  custom colours. Sections are added one at a time in alphabetical order
  (currently Accordion → Toast). Section pattern: `<!-- X Component -->`
  comment, `<h2 class="mt-5">` with the space-padded Bootstrap name (e.g.
  ` List Group `), a reference `<p class="text-body-tertiary">`
  (box-arrow-up-right icon, "Reference:", `<a target="_blank" rel="noopener">`
  with bootstrap icon + `/docs/5.3/components/<name>/` URL), then the demo.
  All sections share the single centered column (no per-section row/col
  wrappers); demo grids use `row mb-5`, and a section's last element carries
  `mb-5`. Headings: h2 per section, h3 per variant, h4 for card titles, h3
  with `fs-5` for modal/offcanvas titles — never skip a level. Ids are unique
  and prefixed per component (`modal-basic`, `toast-live`, ...).
  All three pages link `style/bootstrapail.min.css` in `<head>`, load
  `js/color-modes.js` as `type="module"`, and
  `js/bootstrapail.bundle.min.js` at end of `<body>`. `components/index.html`
  also has an inline `<script>` after the bundle wiring the live Toast demo;
  JS-dependent demos carry a "requires JavaScript" note
  (`<small class="text-body-secondary d-block mt-2">`).
- Layout — all page content follows the centered reading column
  `col-sm-12 col-md-10 col-lg-8 col-xl-6` inside `container-fluid`; use the
  full page width only when explicitly requested.
- Formatting — 2-space indent, no tabs, lowercase tags. Keep lines at ~80
  columns: when a tag would overflow, break before trailing attribute(s) and
  align the continuation with the first attribute; wrap prose at word
  boundaries. Accepted >80 exceptions: reference-link `<a>` blocks, long
  inline colour-swatch spans, and long class-value lines (e.g. the split
  button and the animated progress bar) — never split a class value across
  lines.
- `scss/bootstrapail.scss` — the only style source. Follows Bootstrap 5.3
  "Option B": `@import` functions first, then override the `$primary`,
  `$secondary`, `$success`, `$info`, `$warning`, `$danger` variables, then
  import the rest. Custom CSS goes in the "additional custom code" section
  at the bottom.
- `js/color-modes.js` — hand-maintained light/dark/auto theme toggle
  (adapted from Bootstrap docs); persists choice in `localStorage.theme`.
- `images/sprites.svg` — icon sprite. Add icons here as `<symbol>` elements
  and reference them with `<use xlink:href="images/sprites.svg#icon-id">`.

## Verification

No tests or linters. Verify visually by opening `index.html` in a browser
(after `npm run css`).

---
> Source: [getpail/web-style-guide](https://github.com/getpail/web-style-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
