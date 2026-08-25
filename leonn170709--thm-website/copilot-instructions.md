## thm-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
node server.js        # Start the server (port 3002)
npm install           # Install dependencies (express, basic-auth)
```

No build step, no bundler, no test suite.

## Architecture

Static site served by a minimal Express server (`server.js`). Everything under `public/` is served as-is.

```
public/
  index.html                 # The landing page
  css/thm.css                # The entire design system, shared by every page
  js/thm.js                  # All behaviour, shared by every page
  js/skin.js                 # 3D Minecraft skin viewer, shared by profile pages
  players/<Name>/index.html  # Per-member profile pages
  players/Kyr0zen/art/       # Kyr0zen's portfolio images
  images/                    # .webp assets (banner, logo, map, kitbot icon)
  stats.json                 # Served live via GET /api/stats
  map-meta.json              # Highway map's last-updated timestamp
```

`server.js` does four things: `express.static('public')`, a `GET /api/stats`
endpoint that reads `public/stats.json` with `Cache-Control: no-store` so
Cloudflare can't cache it, `GET /cape/index`, and PNG→WebP. It retries on
`EADDRINUSE` up to 5 times with exponential backoff.

`GET /cape/index` is the cape index for the THM-Addons mod (its `CapeManager`
fetches it as `api.capeIndex`). It answers `{"capes":[{"id","url"}]}`, one entry
per `.webp` in `public/cape/`, `id` being the filename without the extension —
the mod saves each download as `<id>.webp`. Adding a cape is dropping a `.webp`
in that directory; nothing else to edit. The `url` is absolute and built from
`x-forwarded-proto`, since the client is a Java `HttpURLConnection` that won't
follow Cloudflare's http→https redirect.

PNG→WebP: on boot, every `.png` under `public/` gets a lossless `.webp` twin via
system **ffmpeg** (`-c:v libwebp`), skipped when the twin is already newer than
the PNG. A middleware then answers `.png` requests with the twin when the client
sends `Accept: image/webp` and the twin is smaller, `Vary: Accept` set. No image
library in the dependencies; a new PNG converts on the next restart. If ffmpeg is
missing the conversion warns and the PNG is served as-is.

`stats.json` and `map-meta.json` are written by something outside this repo. If
either fetch fails the page degrades quietly (the plate keeps its `—`
placeholders).

## Frontend conventions

No framework, no build. Plain HTML, one shared stylesheet, one shared script.
CSS and JS are **not** inline. Each `<head>` carries exactly two inline
`<script>`s and nothing else: the pre-paint theme setter (which also sets
`style.colorScheme`, so a new document never flashes white), and a
`type="speculationrules"` block — JSON, not behaviour.

### Navigation is client-side

**A same-origin click never loads a document.** `thm.js` intercepts it, fetches
the page, and replaces `<body>` — so nothing blanks, in any browser. Chrome and
Safari wrap the swap in `document.startViewTransition` and crossfade it; Firefox
gets an instant swap. This is the only way to match a site that stays smooth in
Firefox, which has neither view transitions nor speculation rules.

The rule this imposes on all JS: **`thm.js` runs its page wiring more than
once.**

- `initPage()` — everything that touches page content. Runs on load and after
  every swap. It disconnects the previous page's `IntersectionObserver`s first
  (`track()` collects them); anything else stateful must be equally re-entrant.
  The nav is part of the swapped body, so the theme toggle and burger are wired
  here too.
- `initShell()` — window-level listeners only (scroll, the router). Runs once.
- Scripts inside swapped-in markup **never execute**. That is why pages carry no
  `<script>` for three.js or `skin.js`: `skinViewer()` loads them on demand, once
  per session, when a `#skin-canvas` is present. `skin.js` is therefore
  `window.thmSkin()`, not an IIFE, and its render loop stops itself when the
  canvas is detached — otherwise every visited profile leaks a WebGL context.
- Failure hands control back: a bad fetch or an empty document sets
  `location.href`, so a broken page is a normal page load, never a dead end.
- Without JS every link is an ordinary link, which is why the rest still matters.

**No page may show a white frame on a direct load either.** Three pieces, and a
page missing any one of them flashes:

1. **The pre-paint script sets the background**, not just the theme:
   `r.style.background = t === 'light' ? '#f3efe8' : '#0d0b0c'` (the two `--void`
   values). It stays inline — an external file would be a blocking request in
   front of first paint, which is the thing being fixed. `<meta
   name="color-scheme">` and `color-scheme` in `thm.css` back it up but cannot
   replace it: `"dark light"` resolves to **light** on a light desktop and paints
   white even when the visitor picked the dark theme. The theme toggle sets the
   same three properties, or the next swap paints the old theme for a frame.
2. `@view-transition { navigation: auto; }` in `thm.css`, **at top level**, once.
   Not inside `@media` — the opt-in is unreliable there. Reduced motion is
   handled by killing `animation` on `::view-transition-old/new/group(*)`, which
   the global reduced-motion `*` block does not reach.
3. Speculation rules — one document rule per page, `href_matches: "/*"` at
   `"eagerness": "eager"`, plus plain `<link rel="prefetch">`. Both now serve the
   router's `fetch()` out of cache, and cover hard loads (typed URL, middle
   click, no JS). Nothing to maintain when a page is added.

When adding or editing a page, copy an existing `<head>` **whole**, and add no
script tags beyond `/js/thm.js`. Diff against a working page before finishing.

Images all carry `loading="lazy" decoding="async"`; the only eager image is the
22px nav mark. Keep both attributes on anything new.

**Read `DESIGN.md` before touching anything visual.** It is the source of truth
for tokens, type roles, components, motion budget, and the rules the system
holds to. In short:

- Tokens: `--void` / `--basalt` surfaces, `--bone` / `--ash` / `--ash-dim` text,
  `--rule` / `--rule-2` hairlines, `--signal` orange as the one accent.
  Radius `--r-xs` 5 / `--r-sm` 9 / `--r` 14px, `--s2`–`--s9` spacing.
- Fonts (Google Fonts): **Archivo** at expanded widths (display), **Newsreader**
  (body serif), **IBM Plex Mono** (labels and data).
- No glassmorphism, no `backdrop-filter`, no glows or coloured shadows.
- No illustrated highway. The recurring device is the hairline: `hr.seam`
  between sections, `.road` beside the wordmark, and the `.log-rail` that fills
  as you scroll. There is no cross-section drawing — the copy states the width.
- The map is `.net` (text left, `.map` plate right) and opens full size in the
  `#map-zoom` native `<dialog>`; `thm.js` only wires up open/close.
- Every other image enlarges too: `thm.js` builds one `.zoom.lightbox` `<dialog>`
  at load and makes each `<img>` outside `.nav` / `.map-open` / `.zoom` a
  keyboard-reachable zoom trigger. No per-image markup — don't add any.
- One shell: `.wrap` at 1400px. Only `hr.seam` and `.crew` (the full-bleed crew
  render at the head of the Roster) sit outside it — put anything full-width
  outside the `.wrap` div rather than inventing an escape class.
- Scroll reveal: `.rv` elements get `.in` via `IntersectionObserver`. Profile
  pages use the older `.reveal` / `.visible` pair; `thm.js` handles both.
- Every animation respects `prefers-reduced-motion`; content is never gated on a
  JS class.

**Member profile pages** share `thm.css` and `thm.js` and are built from the same
components as the landing page — `.head`, `.h2`, `.prose`, `.plate`, `.facts`,
`.why` — plus a small `.pf-*` set (`.pf`, `.pf-crumb`, `.pf-head`, `.pf-name`,
`.pf-roles`, `.pf-role`, `.pf-bio`, `.pf-plate`, `.pf-body`, `.pf-prose`) and
`.skin-hint` / `#skin-canvas` for the 3D viewer.

When adding a player, copy any existing profile — they are all built from one
template. Keep the font `<link>`, the pre-paint theme script, and
`/js/thm.js?v=8`, which is the page's only script tag.

The skin viewer is `public/js/skin.js`, shared by every profile and configured
from the canvas element, so there is no per-page copy of it. Nothing loads it
from markup — `thm.js` pulls it in with three.js when it sees the canvas:

```html
<canvas id="skin-canvas" data-username="Ryazor" data-slim="false"></canvas>
```

Skins come live from `minotar.net/skin/<username>.png`, with a flat Steve
fallback if the request fails. `data-slim="true"` gives 3px (Alex) arms.

**Roster links:** on the landing page a name is an `<a href="/players/<Name>/">`
only when that page exists; everything else is a plain `<span>` so the roster has
no dead links. When you add a profile, switch its roster entry to a link.

The v2 profile classes are **gone from the CSS**: `.glass`, `.glass-hover`,
`.profile-*`, `.wiki-*`, `.role-badge`, `.facts-table`, `.stat-table`, `.ach-*`
and `.chip`. A page still written against those will render unstyled — port it to
the classes above.

Anchors on the landing page: `#about`, `#log`, `#network`, `#standard`,
`#alliance`, `#roster`. Profile pages link back to `#roster`.

---
> Source: [Leonn170709/THM-Website](https://github.com/Leonn170709/THM-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
