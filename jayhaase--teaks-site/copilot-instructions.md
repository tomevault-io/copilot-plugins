## teaks-site

> Instructions for AI coding agents working in this repo. Read this before

# AGENTS.md

Instructions for AI coding agents working in this repo. Read this before
making changes.

## What this is

A one-page site for Teak, a tree-faery apprentice apothecary character,
built with Eleventy (static site generator), deployed to GitHub Pages via
GitHub Actions. The design follows a handoff doc in `design/` (forest
green/gold palette, Cinzel/Cinzel Decorative/EB Garamond type). Content is
intentionally kept in plain Markdown/JSON files so a non-technical person
can edit it without touching templates or code. Preserve that separation
in any change you make.

## Toolchain

- Package manager is **pnpm** (`packageManager` field in `package.json`
  pins the version) — do not use npm or yarn, do not add a
  `package-lock.json` or `yarn.lock`.
- Node version is pinned in `.nvmrc` (currently 24). Keep `engines.node` in
  `package.json` consistent with it.
- Eleventy config is `eleventy.config.js` (ESM, flat config — this repo is
  `"type": "module"`).

## Commands

```sh
pnpm install         # install deps
pnpm run serve       # dev server with live reload
pnpm run build       # build to _site/
pnpm run lint        # eslint + stylelint + markdownlint
pnpm run format      # prettier --write
pnpm run test        # build + unit + html-validate + a11y (axe) + linkinator
pnpm run validate    # lint + test — this is what CI runs; run it before
                      # considering any change done
```

`_site/` is build output (gitignored) — never hand-edit it, and never
commit it.

`design/` holds design handoff material (mockups, style guides, brand
assets) for reference only — it's listed in `.eleventyignore` and never
gets built or published. Don't wire it into the site.

`scripts/` holds the site's client-side JS (`lightbox.js`, `mischief.js`
— passthrough-copied like `styles/` and `images/`). Each is a plain
browser script (no bundler, no framework, both loaded via separate
`<script defer>` tags in `layout.njk`) — `eslint.config.js` has a
separate block scoped to `scripts/**/*.js` with browser globals instead
of the root config's Node globals. Keep any future client JS in this
folder and pattern for the same reason.

## Content vs. code — don't blur this line

- `content/sections/*.md` and `_data/site.json` are the editable content
  layer. Changes here should only ever require adding/editing a file, not
  touching `_includes/layout.njk`, `index.njk`, or `eleventy.config.js`.
- `content/sections/sections.json` is a directory data file that sets
  `permalink: false` and `layout: false` for every file in that folder.
  Individual section files must **not** need their own `permalink` or
  `layout` frontmatter — if you're adding one, the directory data file is
  broken, fix it there instead.
- Every section file has a `type` field (`about` | `menu` | `gallery` |
  `find-teak`) that picks which branch of the big `{% if %}` chain in
  `index.njk` renders it — that's what lets each section have a bespoke
  layout while staying data-driven. The four types are fixed; adding a
  genuinely new one requires a new template branch, not just a new content
  file (see `README.md`'s "Adding a genuinely new section"). Editing or
  reordering fields/list items _within_ one of the four existing types is
  content work, not code work.
- Don't invent new required frontmatter fields without updating
  `README.md`'s instructions for the non-technical editor.

## Things that will break silently if you're not careful

- **Image `alt` text is mandatory.** The `{% image src, alt, sizes %}`
  shortcode in `eleventy.config.js` throws if `alt` is falsy — that's
  deliberate (accessibility), don't work around it by passing an empty
  string.
- **HTML ids must not start with a digit.** Section files are named
  `01-about.md`, `02-menu.md`, etc. so they sort correctly, and Eleventy's
  `fileSlug` keeps that numeric prefix (e.g. `01-about`) rather than
  stripping it. Templates prefix ids with `section-` (e.g.
  `section-01-about`) to keep them valid — see `index.njk` and
  `_includes/layout.njk`. If you add new id-generating code, keep this
  prefix or html-validate's `valid-id` rule will fail.
- **Nunjucks whitespace control matters here.** `html-validate` runs with
  `no-trailing-whitespace` enabled. Loops/conditionals in `.njk` files use
  `{%-`/`-%}` trim markers on purpose to avoid emitting blank lines. Keep
  that pattern when editing templates.
- The `{% image %}` shortcode writes optimized files to
  `_site/images/optimized/` — source images live in `images/` at the repo
  root, referenced by filename only (no path prefix) from frontmatter/data.
  It takes an optional 4th arg, `focal` (a CSS `object-position` value,
  e.g. `"50% 20%"`), wired through as `style="--focal: ...;"` — CSS reads
  it via `object-position: var(--focal, center)`. There's also an
  `{% imageUrl src %}` shortcode that returns just the largest JPEG URL
  (used by the gallery lightbox's full-size view) — it's called with the
  _same_ `imageOptions` object as `{% image %}` on purpose, so it hits
  eleventy-img's on-disk cache instead of reprocessing the file. If you
  change one shortcode's image options, keep them in sync (or genuinely
  want a different derivative) rather than letting them drift.
- **The gallery grid is deliberately count-agnostic**
  (`repeat(auto-fill, minmax(220px, 1fr))` + `aspect-ratio: 1` per cell) —
  it used to be hand-tuned for exactly 5 photos with one spanning 2 rows,
  which broke as soon as real content had 6. Don't reintroduce a
  fixed-count layout here; the photo list is expected to keep
  growing/shrinking as the site owner adds real photos.
- All asset URLs (stylesheet, favicon, `{% image %}` output) are
  **relative, not root-absolute** (`styles/style.css`, not
  `/styles/style.css`). This is a GitHub Pages _project_ site, served
  under `/teaks-site/`, not domain root — a leading slash silently 404s in
  production while looking fine in local dev. `site.url` in
  `_data/site.json` holds the one place a fully-qualified absolute URL is
  actually required (OG tags, canonical link).
- **CSS specificity can silently defeat a modifier class.** E.g.
  `.gallery-grid figure { border: ... }` (class + element = higher
  specificity) will beat `.gallery-accent { border-color: ... }` (class
  only) regardless of source order, even though the modifier class is
  meant to override it. Verify with `getComputedStyle(...)` in the
  browser/preview tool when adding a variant class, don't just trust that
  stylelint passing means the cascade does what you intended. Prefer a
  CSS custom-property indirection (`border-color: var(--x, var(--fallback))`,
  then set `--x` on the modifier class) to sidestep specificity fights
  entirely — see `.gallery-accent` in `styles/style.css` for the pattern.
- **A plain `1fr` grid track has an implicit `min-width: auto`**, sized to
  its content's min-content — not 0. One long unbreakable string (a faire
  name, an email address) inside a narrow single-column grid is enough to
  blow the whole grid past its container and cause page-wide horizontal
  scroll on mobile, even though the grid's own box is sized correctly.
  Every `grid-template-columns` in `styles/style.css` uses `minmax(0, 1fr)`
  instead of bare `1fr` for this reason — keep that pattern in any new
  grid. Verify with `document.documentElement.scrollWidth` vs
  `clientWidth` at a narrow viewport (should be equal) rather than trusting
  a visual check, since a few px of overflow is easy to miss by eye.
  Long unbreakable button/link text (e.g. an email address) needs its own
  fix — `.btn` has `overflow-wrap: anywhere` for this.
- **A class-based `display` rule beats the `[hidden]` attribute's default
  styling.** The browser's UA stylesheet sets `[hidden] { display: none }`,
  but that's low specificity — `.doodle-toggle img { display: block; }`
  (class + element) overrides it, so a `hidden` toadstool/frog image would
  render anyway, stacked on top of its sibling. Fixed with a same-element,
  higher-specificity rule: `.doodle-toggle img[hidden] { display: none; }`.
  Same family of bug as the two entries above — if you toggle visibility
  via the `hidden` attribute (rather than a class) anywhere, check the
  element doesn't also have an unconditional `display` rule from a class
  selector, and verify with `getComputedStyle(...).display`, not just
  reading the `hidden` attribute back (it can be `true` while the element
  is still visually `block`). Also: the local preview browser tab caches
  `styles/style.css` aggressively — if a CSS fix doesn't seem to take
  effect after editing + rebuilding, hard-reload with a cache-busting query
  string before concluding the fix is wrong.
- `role="button"` on a `<div>` (used for reveal-enabled potion cards) trips
  html-validate's `prefer-native-element` rule, but a real `<button>`
  can't legally contain block content like `<h3>`/`<p>` per the HTML5
  content model (`element-permitted-content`) — the two rules directly
  contradict for "clickable card with rich content," a legitimate and
  ARIA-sanctioned pattern. `prefer-native-element` is turned off in
  `.htmlvalidate.json` for this reason. Cards using this pattern need
  `tabindex="0"` plus a JS `keydown` handler for Enter/Space — native
  buttons get that for free, `role="button"` divs don't.

## CI/CD — don't weaken the safety gate

`.github/workflows/ci.yml` has two jobs: `test` (runs on every push and PR)
and `deploy` (`needs: test`, only runs on push to `main`). The whole point
is that a broken build can never reach GitHub Pages. If you touch this
file:

- Keep `deploy` gated behind `needs: test`.
- Keep it scoped to `github.ref == 'refs/heads/main'` for the deploy job.
- Don't add `continue-on-error` to any of the lint/test steps.

`.github/dependabot.yml` sets `cooldown.default-days: 3` for both the npm
and github-actions ecosystems on purpose (supply-chain safeguard — a newly
published package version must be public for 3 days before Dependabot will
open a PR for it). Don't remove this without being asked.

## Before finishing any task

Run `pnpm run validate`. It must pass — this is exactly what CI enforces,
so if it fails locally the PR will fail too.

---
> Source: [jayhaase/teaks-site](https://github.com/jayhaase/teaks-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
