## field-manual-engine

> Generate a complete interactive field-manual website about ANY topic (espresso, options trading, Roman history, music theory, …). Trigger on "interactive manual", "field manual", "interactive guide about X", "interactive encyclopedia about X", "interactive handbook about X", "teach X as an interactive site", or "build me a manual/encyclopedia like the AI Encyclopedia". Produces a Palantir-dark × Apple-restraint static multi-page site with live instruments, hover glossary, optional KaTeX math and runnable Python — the same engine that powers the AI Encyclopedia (llm-manual.vercel.app).


# THE FIELD MANUAL ENGINE

You are about to build an interactive field manual that is **indistinguishable in
craft from the AI Encyclopedia** (llm-manual.vercel.app) — about whatever topic the
user names. The engine is topic-agnostic; the craft rules are not negotiable.

Before doing anything else, read these files from this skill's directory:

1. `engine/AUTHORING.md` — the per-chapter spec (components, voice, self-checks)
2. `engine/templates/chapter-template.html` — the chapter skeleton
3. `engine/templates/index-template.html` — the hub skeleton
4. `engine/assets/js/GLOSSARY-NOTE.md` — how to swap the glossary per topic

Skim `engine/assets/css/manual.css` (the design system) and `engine/assets/js/shared.js`
(the FM helper API) so you know what exists before inventing anything.

---

## A · INTAKE — ask before you build

Ask the user these, compactly, in one message. If they say "just build it," use the
defaults in brackets and proceed without further questions.

1. **Topic & scope edge** — what's in, what's out? ("espresso" = the drink + dialing
   in + machines? or also roasting and café business?) [you draw the line, state it]
2. **Audience & depth** — curious beginner, serious hobbyist, working practitioner,
   or a mix? [mix: arc starts INTRO, ends ADVANCED]
3. **Chapter count** — 6–12 chapters + capstone. [propose 8 + capstone]
4. **Accent preset** — show the five pairs from section H, recommend the one that
   fits the topic's temperament. [your recommendation]
5. **Deploy target** — Vercel, GitHub Pages, or just local? [local + ready-to-deploy]

Then, **before writing any HTML**, post a one-screen plan and get a nod:
the chapter arc (titles + one-line theses + level badges), the topic's
quantities-with-knobs and process-worth-simulating (section C), and the accent
choice. This plan is cheap; rebuilt chapters are not.

---

## B · THE PEDAGOGY ARC

Every manual follows the same five-act arc, scaled to the chapter count. Each
chapter gets a difficulty badge: **INTRO** (no prerequisites beyond curiosity),
**CORE** (comfortable with the manual's notation/canon), **ADVANCED**
(practitioner-adjacent).

| Act | Share | What it does | Badge drift |
|---|---|---|---|
| **Foundations** | ~25% | The atoms of the domain: definitions, units, the one core loop or object everything else builds on. Reader leaves able to *name things precisely*. | INTRO |
| **Mechanics** | ~25% | How the atoms interact — the central mechanism opened up and played with. The manual's densest instruments live here. | INTRO → CORE |
| **Systems** | ~25% | Composition at scale: workflows, trade-offs, failure modes, "the bill." Where the calculators and trade-off frontiers live. | CORE |
| **Frontier** | ~15% | Live debates, recent developments, what experts disagree about, what's unsolved. Honest hedging is content here, not weakness. | ADVANCED |
| **Capstone** | 1 chapter | A synthesis instrument: the reader *configures something end-to-end* with live numbers (a dossier/result card), plus 3–5 drill cards that grade instantly. | CORE |

Example mappings (so this never collapses into vagueness):

- **Espresso (8+1):** what coffee is → grind & water chemistry → the shot
  (pressure/flow/extraction) → dialing in → milk → machines → sourcing & roast
  levels → the frontier (turbo shots, profiling debates) → capstone: design your
  bar + diagnose five broken shots.
- **Options trading (10+1):** contracts & payoffs → pricing intuition →
  Black–Scholes & the Greeks → volatility → spreads → income structures → risk &
  sizing → market microstructure → frontier (0DTE, vol products) → capstone:
  build a position, watch it age through a vol shock.
- **Roman history (9+1):** geography & sources → kingdom → republic machinery →
  the army → the revolution century → principate → economy & daily life → crisis
  & dominate → fall debates → capstone: the empire timeline scrubber + drills.

Chapter ordering rule: each chapter's hero lists `BUILDS ON` — the arc must be a
DAG that never points forward.

---

## C · THE INSTRUMENT ARCHETYPE LIBRARY

**This is the heart of topic-agnosticism.** Before scaffolding, do the mapping:

> **List the topic's 3–5 quantities-with-knobs** — continuous variables an expert
> actually reasons over (espresso: dose, grind setting, water temp, pressure, time;
> options: strike, days-to-expiry, implied vol, rate; Rome: year, army size, grain
> supply; music theory: tempo, interval, scale degree, voice count).
> **And ONE process-worth-simulating** — the multi-step temporal thing at the
> domain's center (the 30-second shot; an option's life through theta decay; a
> campaign season; a 12-bar progression resolving).

Then assign archetypes: quantity-clusters → 1/3/7/8, the process → 2/5, structure →
4/9, judgment → 6, retention → 10. Every chapter gets ≥2 instruments from the
mapped set; the same archetype may recur with different content but never with the
same visual layout twice in adjacent chapters.

All sketches below assume the widget HTML pattern from `engine/AUTHORING.md` and the
FM helpers (`FM.setupCanvas`, `FM.softmax`, `FM.mulberry32`, `FM.siFormat`, `FM.C`).

### 1 · PARAMETER EXPLORER
**When:** one or two knobs visibly reshape a curve or surface. The default first
instrument of any quantitative chapter.
**Sketch:** `.ctl-row` of range sliders → `render()` recomputes a closed-form model
→ redraw curve on `canvas.w-canvas` via `FM.setupCanvas` (axes in `FM.C.HAIR`,
primary curve `FM.C.MINT`, comparison `FM.C.BLUE`) → update 2–4 `.readout` values
with `FM.siFormat`. Re-render on `input` and `resize`. ~60 lines.
*Flagship example: the LoRA parameter counter; espresso: grind size → extraction-yield curve.*

### 2 · PROCESS SIMULATOR
**When:** the domain's central process unfolds in steps/phases and seeing it run
beats reading about it.
**Sketch:** state object + `step()` function; PLAY/PAUSE/STEP/RESET `.w-btn`s;
`setInterval(step, 350)` (NEVER rAF chains — section F); `.stage-strip` showing the
current phase with `.on`; canvas plots the evolving trace. End state stays on
screen. ~120 lines.
*Flagship: k-means stepper, token journey; espresso: the 30-second shot — pre-infusion → ramp → decline, flow & yield accumulating live.*

### 3 · TRADE-OFF FRONTIER
**When:** two desiderata fight (quality vs cost, speed vs safety, body vs clarity)
and the domain has a Pareto edge.
**Sketch:** canvas scatter of 30–80 candidate configurations from a seeded
`FM.mulberry32` generator or a real table; frontier polyline in `FM.C.MINT`;
dominated points dimmed; a slider or click moves "you are here" and readouts show
what you give up in each direction. ~90 lines.
*Flagship: roofline model; options: risk vs premium across strikes.*

### 4 · ANATOMY PANEL
**When:** a physical or structural object has named parts worth knowing (machine,
contract, legion, sonata form).
**Sketch:** `.figure` inline SVG of the object built from `dg-box`/`dg-line`
primitives; each part is a `<g data-part="name" cursor:pointer>`; click toggles the
part to `dg-box-accent` and swaps an explainer `<div>` below the figure (plain DOM,
no canvas). Initial state: first part pre-selected. ~70 lines.
*Flagship: transformer block diagram; espresso: portafilter-to-boiler cutaway.*

### 5 · TIMELINE SCRUBBER
**When:** the domain has history or any long axis (years, eras, frets, geological
layers) where state-at-time-t is the lesson.
**Sketch:** one wide range input over the axis; a `DATA` array of `{t, label,
metrics, note}` keyframes; `render(t)` interpolates metrics, redraws a canvas strip
(area chart, map-like blocks, or bar race) and updates a caption + readouts. Add
`.tok-chip` markers for landmark events that jump the slider on click. ~100 lines.
*Rome: empire extent & legion count by year, 500 BCE – 476 CE.*

### 6 · DECISION-TREE WALKTHROUGH
**When:** practitioners follow a real diagnostic or selection procedure ("shot
tastes sour → ?", "which spread fits this view?").
**Sketch:** DOM-only state machine: a question card (`.drill`-style panel) with
2–4 `.w-btn ghost` options; chosen path renders as breadcrumb `.tok-chip`s; leaves
are a recommendation `.callout` with a RESTART button. Encode the tree as a nested
object literal; ~15 nodes max. ~80 lines.
*Flagship: "to tune or not to tune"; espresso: the dial-in flowchart.*

### 7 · DISTRIBUTION / SAMPLING TOY
**When:** the domain has randomness, populations, or repeated trials whose *shape*
matters (returns, cupping scores, dice mechanics, measurement error).
**Sketch:** `FM.mulberry32(seed)` draws N samples from the domain's model;
histogram on canvas (bars `FM.C.MINT`, overlay theoretical curve `FM.C.BLUE`);
sliders for the distribution's 1–2 parameters; a RESAMPLE button re-seeds; readouts
for mean/spread/tail stat. `FM.softmax` with a temperature slider is the ready-made
"sharpen/flatten preferences" toy for any choice-among-options story. ~90 lines.
*Options: P&L distribution of a strategy across 1,000 seeded price paths.*

### 8 · CALCULATOR WITH READOUT STRIP
**When:** the domain has a formula or recipe practitioners actually punch numbers
into. The cheapest archetype — every manual should ship at least two.
**Sketch:** no canvas. `.ctl-row` of sliders/`.w-input`s → pure function → 3–5
`.readout`s (`.acc` on the headline number, `FM.siFormat` for big values). Add a
`.w-btn ghost` seg-row for discrete modes. If a threshold matters (fits/doesn't,
profitable/not), color the readout `FM.C.RED` via inline style when crossed. ~40 lines.
*Flagship: VRAM "will it fine-tune?"; espresso: brew-ratio & dose calculator.*

### 9 · COMPARISON MATRIX
**When:** 4–8 named alternatives differ along 3–6 dimensions (brew methods, spread
types, emperors, scale modes).
**Sketch:** `table.data` with a control row of `.w-btn ghost` toggles ("sort by X",
"highlight what fits constraint Y"); clicking re-sorts rows (detach/re-append
`<tr>`s) and applies `td.hl` to winning cells. Or: two `.spec-grid` cards
side-by-side with a selector swapping the right-hand card. DOM-only. ~60 lines.
*Flagship: the PEFT zoo; music: mode comparison with sounding characteristics.*

### 10 · DRILL CARD
**When:** capstone, and sparingly at the end of dense CORE chapters. Retention is
a feature.
**Sketch:** the `.drill` pattern verbatim (manual.css ships it): question `.dr-q`,
option buttons `.dr-opt`, on click add `.right`/`.wrong`, reveal `.dr-exp.show`
with the explanation, disable further clicks. 3–5 per capstone, each testing a
different chapter's bold idea. ~30 lines each.

---

## D · EQUATIONS WHERE THEY EXIST — AND ONLY THERE

- **Quantitative domains** (trading, brewing physics, acoustics, navigation,
  photography exposure…): KaTeX everywhere it earns its place. Display math in
  `.eq` blocks with `eq-tag` numbering (`EQ {ch}.{n}`); inline `\( ... \)`. A
  notation appendix on the hub (the `table.data` pattern).
- **Qualitative domains** (history, mythology, design movements…): **delete the
  KaTeX `<link>` and both KaTeX `<script>` tags from every page.** The structural
  spine is instead `.figure` SVG **frameworks** with `fig-id` numbering
  (`FRAMEWORK {ch}.{n}`) — named diagrams of forces, factions, timelines, causal
  arrows. They occupy the same visual register as equations and are referenced
  from prose the same way. The hub's notation appendix becomes a **Canon** table:
  the 10–15 terms the manual uses in a fixed sense.
- Never fake rigor: no decorative Greek letters in a history manual, no equation
  restated in prose immediately after (the `.eq-note` does that job).
- Mixed domains (music theory has both) use both, per chapter, honestly.

---

## E · ENGINE MANIFEST & SCAFFOLDING

Target layout for a generated manual (e.g. `espresso-manual/`):

```
espresso-manual/
├── index.html                  ← from engine/templates/index-template.html
├── chapters/
│   ├── 01-the-bean.html        ← from engine/templates/chapter-template.html
│   ├── 02-….html …
│   └── capstone.html
├── assets/
│   ├── css/manual.css          ← copied VERBATIM, then themed (section H)
│   └── js/
│       ├── shared.js           ← VERBATIM (FM helpers, GSAP choreography, scrollspy, progress)
│       ├── glossary.js         ← VERBATIM except the DEFS dictionary (GLOSSARY-NOTE.md)
│       ├── search.js           ← optional; needs /search-index.json (below)
│       └── pyrunner.js         ← only if any chapter uses .pycell
├── search-index.json           ← optional, hand- or script-built
└── vercel.json                 ← if deploying to Vercel (section J)
```

Scaffolding steps, in order:

1. `mkdir -p <manual>/chapters <manual>/assets/css <manual>/assets/js`
2. `cp` the assets from this skill's `engine/assets/` into place. **Copy, never
   symlink, never retype** — these files are the engine.
3. Apply the accent theme to the copies (section H). Do this BEFORE writing
   chapters so canvas screenshots match from the start.
4. Replace `DEFS` in the copied `glossary.js` per `GLOSSARY-NOTE.md` (40–120 terms;
   write the first ~40 from the chapter plan now, merge per-chapter additions later).
5. Write `index.html` from the hub template: pure-CSS animated cover (the template's
   `.cover-bg` — no Three.js), `toc-list` of all chapters with one-line hooks +
   level badges + instrument tags, footer with the attribution block (section I).
6. Write chapters from the chapter template, each obeying `engine/AUTHORING.md`
   end to end. **All chapter-specific widget JS goes inline in that chapter's
   single IIFE** — shared files are never edited after step 4.
   Chapters parallelize well: if you fan out subagents, give each the brief
   (chapter thesis, sections, assigned archetypes, prev/next filenames, slug
   prefix) + `engine/AUTHORING.md`, and have each return its 3–8 glossary terms
   in its report for you to merge.
7. Optional search palette: build `search-index.json` at site root — a JSON array
   of `{"t": title, "s": subtitle/first-words, "b": body text (first ~200 words),
   "u": "/chapters/01-….html", "v": "CH 01"}` — one entry per page. search.js
   fetches it from the root-absolute path `/search-index.json`.
8. Keep chapter files under `chapters/` — shared.js marks reading progress only
   for paths matching `/(chapters|ml|prompting|agents)/`. If you must use another
   directory name, updating that one regex in YOUR COPY of shared.js at scaffold
   time is the single allowed shared-file edit.

---

## F · HARD-WON GOTCHAS (each cost a debugging session — verbatim, do not relearn)

1. **GLSL reserved words.** If anyone adds a WebGL flourish: `active` is a reserved
   word in GLSL — using it as an attribute/varying name fails with an opaque shader
   compile error. (The engine's templates avoid WebGL entirely; the pure-CSS cover
   exists partly because of this class of bug.)
2. **KaTeX reflow breaks anchors and triggers.** Math rendering changes content
   heights after load, so `#s4` links land in the wrong place and ScrollTrigger
   positions go stale. shared.js fixes both — re-scrolls to `location.hash` and
   calls `ScrollTrigger.refresh()` after render — but ONLY if the auto-render
   script tag keeps its hook: `onload="if(window.__manualMathReady)window.__manualMathReady()"`.
   Never strip that attribute.
3. **setTimeout, not requestAnimationFrame, for multi-step animations.** Background
   tabs throttle rAF to zero; an rAF-chained simulation freezes mid-run and the
   reader returns to a half-drawn widget. `setInterval`/`setTimeout` keep ticking.
4. **Cache busting.** When iterating on a shared asset mid-session, bump a query
   string on its tag (`shared.js?v=2`) or you will debug a stale file. The shipped
   `vercel.json` caches `/assets/` for one hour — same trap in production.
5. **Unique, slug-prefixed ids.** Every element id on a page is prefixed with the
   chapter slug (`shot-canvas`, `shot-pressure`). Duplicate ids fail silently:
   `getElementById` returns the first match and the second widget just doesn't work.
6. **Palette discipline.** Only the locked palette + rgba tints of it. New "helpful"
   colors (a yellow warning here, a purple chart there) are the fastest way to lose
   the Palantir-grade look. In canvas code use `FM.C.*` only; in markup use CSS vars.
7. **Reduced motion.** `prefers-reduced-motion` collapses all CSS animation
   (manual.css does this globally) and shared.js skips GSAP entirely — so content
   must never be gated behind an animation, and every widget must render a complete,
   correct initial state with zero interaction.
8. **file:// does not work.** `fetch()` of the search index and ES-module imports
   both fail off the filesystem. Always serve: `python3 -m http.server 4173`.
9. **Escape `<` as `&lt;` inside KaTeX source** — a bare `<` followed by a letter
   is parsed as an HTML tag opening and silently eats the rest of the equation.

---

## G · QA CHECKLIST — the smoke test is a SHIP GATE

Per chapter, run the self-check list at the bottom of `engine/AUTHORING.md` (links,
ids, IIFE hygiene, math, widget defaults).

Then the site-level pass — **a manual does not ship until every line passes**:

1. `python3 -m http.server 4173` in the manual root; open every page over http.
2. Zero console errors on every page (including 404s for assets — check the
   network tab once).
3. Every widget shows a meaningful initial state before any interaction; every
   control changes something visible.
4. Hub: every `toc-item` href resolves; chapter count claims match reality.
5. Pager chain: walk NEXT from chapter 01 to capstone without a dead link; walk
   PREVIOUS back.
6. Glossary tooltips appear on hover in prose and NOT inside headings/code/widgets.
7. Resize to 380 px width: no horizontal scroll, side-nav hidden, pager stacks,
   widgets usable.
8. Emulate `prefers-reduced-motion`: all content visible, nothing waiting on an
   animation.
9. If KaTeX is in use: pick the math-heaviest page, confirm no red "ParseError"
   text and that a `#sN` deep link lands on the right section after load.
10. If search.js shipped: ⌘K opens, a known term finds its page, ESC closes.
11. Read one full chapter aloud-in-your-head for placeholder leakage: no `{{`, no
    "TODO", no lorem. `grep -rn "{{" <manual>/` must return nothing.

---

## H · THEMING — accent pairs

The engine's only theming axis is the accent pair (`--accent` bright /
`--accent-deep` dark-of-same-hue) plus their rgba tints. Everything else is locked.

Five curated presets:

| Preset | Fits | `--accent` | `--accent-deep` | accent RGB | deep RGB |
|---|---|---|---|---|---|
| **REACTOR MINT** (default) | tech, science, the flagship look | `#a6f2cc` | `#2b5945` | `166, 242, 204` | `43, 89, 69` |
| **ROASTED AMBER** | espresso, whisky, woodcraft, baking | `#f2cfa6` | `#59452b` | `242, 207, 166` | `89, 69, 43` |
| **ALTIMETER ICE** | aviation, finance, oceanography, alpinism | `#a6d8f2` | `#2b4759` | `166, 216, 242` | `43, 71, 89` |
| **NOCTURNE VIOLET** | music theory, mythology, astronomy | `#c9a6f2` | `#3f2b59` | `201, 166, 242` | `63, 43, 89` |
| **FORGE EMBER** | Roman history, metallurgy, volcanology, BBQ | `#f2b3a6` | `#592f2b` | `242, 179, 166` | `89, 47, 43` |

Application recipe (the accent literals appear in `manual.css` AND in `shared.js`'s
`FM.C` — replace in the manual's copied assets, never in this skill's originals).
Example, ROASTED AMBER on macOS sed:

```bash
cd <manual>/assets
sed -i '' -e 's/#a6f2cc/#f2cfa6/g' -e 's/#2b5945/#59452b/g' \
  -e 's/166, 242, 204/242, 207, 166/g' -e 's/166,242,204/242,207,166/g' \
  -e 's/43, 89, 69/89, 69, 43/g' -e 's/43,89,69/89,69,43/g' \
  css/manual.css js/shared.js
```

(Linux: `sed -i` without `''`.) After theming, `grep -rn "a6f2cc" <manual>/assets`
must return nothing. `--blue #4e8af7` stays as the universal comparison color and
`--danger #ff4136` as the universal alarm — do not theme those. Custom pairs are
allowed if the user insists: pick a bright tint around L*≈90 and a deep shade of
the same hue around L*≈35, then run the same six substitutions.

---

## I · ATTRIBUTION — default ON

Every generated manual ships with this footer block on the hub AND in chapter
footers (both templates already contain it):

```html
<a href="https://github.com/Emposes/field-manual-engine" target="_blank" rel="noopener">BUILT WITH THE FIELD MANUAL ENGINE ↗</a>
<a href="https://llm-manual.vercel.app" target="_blank" rel="noopener">FLAGSHIP: THE AI ENCYCLOPEDIA ↗</a>
```

Keep it unless the user explicitly asks for removal. If they do ask, remove it
without argument — it's their site.

---

## J · DEPLOY

**Local (always, and the QA gate):**
```bash
cd <manual> && python3 -m http.server 4173   # → http://localhost:4173
```

**Vercel (recommended — the flagship lives there):** drop this `vercel.json` in the
manual root, then `npx vercel deploy --prod --yes`:
```json
{
  "cleanUrls": true,
  "trailingSlash": false,
  "headers": [
    { "source": "/assets/(.*)",
      "headers": [ { "key": "Cache-Control", "value": "public, max-age=3600, must-revalidate" } ] }
  ]
}
```

**GitHub Pages:** push, enable Pages on the branch — it's pure static HTML, no
build step. One trap: search.js fetches the root-absolute `/search-index.json`,
and project pages serve under `/<repo>/`. Either deploy as a user/custom-domain
site, or change `INDEX_URL` in your copy of search.js to a relative-from-root path
at scaffold time, or skip the search palette.

---

## QUALITY BAR, RESTATED

The reader should not be able to tell whether your manual or the AI Encyclopedia
was the original. That means: every chapter fully written with real expertise
(research the topic properly — no padding, no hedging-by-vagueness), ≥2 live
instruments per chapter that teach rather than decorate, a glossary that catches
the terms a newcomer actually trips on, one bold idea per lede, honest frontier
chapters, and a capstone the reader wants to screenshot. If a chapter would ship
thinner than the flagship's Chapter 06, it is not done.

---
> Source: [Emposes/field-manual-engine](https://github.com/Emposes/field-manual-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
