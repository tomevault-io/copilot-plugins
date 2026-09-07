## dooleys-2

> Marketing page for **The Grand**, a new events venue by DOOLEYS in Lidcombe, NSW (opening early 2027).

# The Grand — Modern Grand concept

Marketing page for **The Grand**, a new events venue by DOOLEYS in Lidcombe, NSW (opening early 2027).
This is a **static, no-build site**: plain HTML + CSS + JS, no bundler, no framework, no package.json.
Open `index.html` directly or serve with `python3 -m http.server 8000`.

## Purpose

This build exists to be imported into Figma via the **html.to.design** plugin (File tab → drop the zip),
so the design can be iterated on as editable Figma layers. Anything that breaks a clean static render
breaks the import — avoid adding runtime-fetched content, framework scaffolding, or build steps.

## Structure

```
index.html      single page, all sections inline
styles.css      single stylesheet, minified-ish (one long line per section, grouped by concern)
script.js       progressive enhancement only — the page renders fully without it
assets/         7 images (6 jpg + 1 svg logo)
assets/fonts/   PPEiko-LightItalic.woff2 (self-hosted)
reference/      baseline screenshots at 1440 / 834 / 390
```

## Origin — read this before editing CSS

This page was **extracted from a three-concept demo file** (`DOOLEYS-grand-demo-230826`) where
Modern Grand, Living Celebration and Quiet Luxury all shared one page, switched by a
`data-concept` attribute on `<html>`. The extraction removed the other two concepts, the demo
selector bar, and the `--demo-height` offset, then **flattened** all
`html[data-concept="grand"] …` selectors down to plain selectors.

**The gotcha this created:** flattening removed `(0,1,1)` of specificity from every Modern Grand rule.
Most still won on source order, but one did not — the inverted first package card's body copy
(`.package-card:first-child p`) fell *below* the base `.package-card div>p:last-child` rule and
reverted to `--muted`. It's now scoped as `.packages .package-card:first-child` to win.

If you add a rule that needs to beat a base rule, **check the specificity maths** rather than
assuming source order saves you. The base rules use descendant/child combinators fairly liberally.

The extraction was verified pixel-perfect (0px diff) against the original at all three breakpoints.
Keep it that way — see Verification below.

## Typography

| Role | Face | Source |
|---|---|---|
| Sans / body / UI | DM Sans 400·500·600 | Google Fonts |
| Display serif | PP Eiko Light Italic | self-hosted woff2 |

- `--serif` is **italic-only**. There is exactly one PP Eiko file. It's declared
  `font-weight:100 900; font-style:italic` so any weight request maps to it without synthetic bolding.
- Every `var(--serif)` usage is italic at weight 300. If you introduce an upright serif usage it will
  fall back to Georgia — don't, unless you're also adding a roman file.
- PP Eiko replaced Newsreader. It has a smaller x-height and narrower set width, so it reads slightly
  lighter at the same size. An open question: whether `h1 em, h2 em` wants ~`1.06em` to restore optical
  parity with the roman. Not applied — deliberate design decision left to the designer.
- PP Eiko is **not on Google Fonts**. It must be installed locally for the Figma import to map it.

## Layout system

- Breakpoints that genuinely restructure: **1050px**, **900px**, **760px**.
- The events grid changes column spans at 900 and 760 — the area most likely to regress from a
  careless edit. The packages grid is now a plain `repeat(3,1fr)` with six equal tiles (2 cols at
  1050, 1 col at 760); no card spans multiple columns any more.
- `.hero__shape` (the orbiting arc) is desktop-only, hidden `!important` below 900px.
- Colour: `--ink` `#18302b`, `--paper` `#f6f2ea`, gold accent on CTAs. Flat colour, no gradients
  except the header scrim.
- **`.offer` uses `background-attachment:fixed`** on desktop. A full-page screenshot renders fixed
  backgrounds against the whole page rather than the viewport, so the offer band looks lighter and
  more mottled in captures than it does in a real browser. Don't "correct" the overlay opacity on
  the basis of a screenshot — measure the band median instead.
- **Claret `#750b22`** is the accent introduced with the comps: it sets the events card headings
  and the ground of the first packages tile. If more claret appears, promote it to a variable
  rather than repeating the hex.
- The **events section overrides the global palette**: warm ground `#dbd2bf`, cream tiles `#fffdf7`,
  claret card headings `#750b22`. These are scoped to `.events` only — the other `section--tint`
  sections (location, FAQ) still use the pale green `--tint`. If the warm treatment is meant to be
  global, change `--tint` rather than adding more scoped overrides.

## JS behaviour

`script.js` is progressive enhancement only — header scroll state, mobile nav toggle, enquiry-form
prefill from CTA clicks, IntersectionObserver reveals, and the language dropdown.

The language selector is wired to **GTranslate**, gated by `TRANSLATION_ENABLED` in `script.js`:
the engine loads only when the `.gtranslate_wrapper` host element exists *and* the page is on an
http(s) origin. See `SERVE.md`. Consequences:

- Over `file://` the selector opens and updates its label but does not translate. That is expected,
  not a bug.
- The **Figma-import build omits the host element** on purpose. GTranslate reloads the page to
  apply a language, which corrupts an html.to.design capture. Don't add it back to that build.
- Selecting English is a cookie-expiry plus full reload, not a `doGTranslate` call — Google has no
  "translate back to source" call. Scroll position is stashed in `sessionStorage` across it.
- Changing the language list means editing **both** `gtranslateSettings.languages` and the
  `#language-menu` buttons.

`.motion-reveal` elements start at `opacity:0` and transition over **700ms**. Any screenshot script
must either wait longer than that or disable transitions, or it will capture mid-fade and produce
false diffs.

Images use `loading="lazy"` and a full-page screenshot does **not** reliably paint them — scrolling
the page first is not sufficient either. `scripts/shoot.py` sets every image to `loading="eager"`
and awaits `img.decode()` before capturing; don't remove that step or lower sections will
screenshot blank and silently corrupt the baseline.

**`img` width/height attributes map to CSS presentational hints.** That means a `height` attribute
suppresses `aspect-ratio` unless the author CSS also sets `height:auto`. `.spaces__image-wrap img`
hit exactly this. If you add an image whose box is meant to be governed by `aspect-ratio`, set
`height:auto` alongside it.

## Verification

`scripts/shoot.py` renders the page at 1440 / 834 / 390 and writes full-page screenshots.
`scripts/diff.py` pixel-diffs two screenshot sets.

**`reference/` is gitignored and not in the repo.** Capture a baseline before editing —
`python3 scripts/shoot.py reference` — otherwise there is nothing to diff against. It was
removed from history because eight regenerated copies of a 3-5MB PNG had grown `.git` to 80MB.

```bash
pip install playwright pillow && python3 -m playwright install chromium
python3 scripts/shoot.py out/after
python3 scripts/diff.py reference out/after
```

Before/after diffing is the fastest way to confirm a change did only what you intended. When
diffing, **disable transitions and animations** (the scripts already inject
`*{transition:none!important;animation:none!important}`) — otherwise the reveal fades and the
map-pin pulse produce noise that looks like real regressions.

Expect zero diff outside the region you touched. Anti-aliasing noise floor on identical renders is
~250px on desktop; anything materially above that in an untouched region is a real regression.

## Conventions

- Keep `styles.css` in its existing dense format — one logical section per line, grouped by concern.
  Don't reformat the whole file; it makes diffs unreadable.
- No build step. No dependencies in the page itself.
- Images live in `assets/` with relative paths. Keep them relative — absolute or remote paths break
  the zip-based Figma import.
- `IMAGE-SOURCES.md` tracks provenance for the photography. Update it if images change.

---
> Source: [cohan-ui/Dooleys-2](https://github.com/cohan-ui/Dooleys-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
