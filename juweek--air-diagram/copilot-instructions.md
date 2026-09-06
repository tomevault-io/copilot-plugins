## air-diagram

> A single-purpose data-tool page. You search a ZIP or city (or pick a scenario)

# What's in the Air — agent handoff

A single-purpose data-tool page. You search a ZIP or city (or pick a scenario)
and it draws that air **particle by particle** with a live p5 canvas, next to a
readout that makes one editorial point:

> The green "Good" badge hides two things — a **legal line that's far looser
> than the health line**, and **what's actually in a breath**.

It's built on the Gourmet Data project template (see `README.md` for the generic
template docs). This file is the project-specific map: how it's wired, and the
priorities that should break ties when you change it.

---

## Priorities (read these first — they decide design questions)

1. **Simplicity & maintainability.** Idiomatic React/JS, plain functions over
   clever abstractions. This is a small site; keep it small. A new data source,
   viz, or scenario should be *one file following a convention*, not an edit to a
   central engine.
2. **Honesty about the data.** This is the whole soul of the piece. The official
   PM2.5 number is a single mass figure — it does **not** say what the particles
   are or where they came from, and ultrafine particles aren't in it at all. So:
   - The source split and the ultrafine swarm are **modeled, not measured**, and
     the UI must always say so (`site.config.js` attribution note, the legend
     footnotes). Never let a modeled number read as a measurement.
   - **Zero must mean zero.** If an input pollutant is absent, its source must
     read ~0 — no artificial baselines conjuring particles. (We already fixed
     one over-inclusion bug here; don't reintroduce additive floors in
     `composition.js`.)
   - Scenarios (cigarette, wildfire, …) use **illustrative published
     concentrations**, clearly labeled as such, not live data.
3. **Clarity of copy.** The reader should never be confused about what a number
   means. Percentages sum to 100 over the regulated sources; ultrafine is shown
   as an extra count *outside* that 100 with an explanation; pollutants are
   framed against the bulk gases (a breath is ~99.9% N₂/O₂/Ar). When in doubt,
   explain more.
4. **The narrative comes through the visuals.** The gap between the **US legal
   line** and the **WHO health line** should be visible, not just stated — e.g.
   the by-pollutant view shows both lines at once; the two ring canvases (WHO
   dense vs legal sparse) show the same air judged two ways.
5. **Polish & containment.** Nothing escapes its parent bounds; works at mobile
   width; active controls are unmistakable (solid black); the canvas stays
   performant. The owner iterates **visually** — verify changes in the browser
   and share screenshots, don't just assert they work.
6. **Every state is a URL.** `/Detroit`, `/90001`, `/cigarette` are all real,
   shareable, embeddable addresses. Keep it that way; don't move result state
   into component-only state.

---

## Architecture at a glance

```
URL (/:query)
   └─ AirPage  ──useAsync(getByQuery, query)──▶  getByQuery(query)
        │                                          ├─ getScenario()  → preset (no network)
        │                                          └─ geocode() + fetchAirQuality() + nowcastAqi()
        │                                                    ↓
        │                        { location, current, nowcast, blurb?, source? }
        │
        ├─ sketchData (memoized)  ─▶  <P5Sketch sketch={airParticleSketch} />   (the canvas)
        └─ <Readout>              ─▶  AQI meter + per-view legend                (the text column)
```

- **One page, three URLs, one component.** `App.jsx` routes `/`, `/:query`, and
  `*` all to `AirPage`. The catch-all keeps bad links rendering the app, not a
  404. `basename` supports subpath embeds.
- **Async is one ~30-line hook.** `lib/useAsync.js` runs `getByQuery(key)` when
  the URL key changes, with a per-key in-memory cache and
  `idle|loading|done|error` status. Pass it a **stable module-level function**
  (it is — `getByQuery`), never an inline arrow.
- **View/mode/toggles are local state**, deliberately *not* part of the fetch
  key, so flipping views never refetches.

### Data layer (`src/data/`, `src/lib/`)

| File | Responsibility |
|---|---|
| `data/airQuality.js` | `getByQuery` (the one entry point) → geocode + fetch + NowCast. Geocoding: ZIP via Zippopotam.us, city via Open-Meteo geocoder. Air data via Open-Meteo air-quality API. **All three are free, CORS-open, no API key** — the browser calls them directly, no serverless proxy. |
| `data/scenarios.js` | `SCENARIOS` presets + `getScenario(query)`. `getByQuery` checks this first, so `/cigarette` short-circuits the network and returns a `current`-shaped object plus a `blurb` and its own `source`. Values are illustrative literature figures. |
| `lib/pollutants.js` | `POLLUTANTS` (one entry per API field: range, `who` line, `legal` line, color), `PM25_LINES`, `exceedance(def,value,mode)`, `aqiCategory(aqi)`. The WHO/legal gap lives here. |
| `lib/composition.js` | The **modeled** source split. `composition()` apportions PM2.5 mass across soot/brake/haze/wildfire/bio using the other pollutants as proxies; `particleBreakdown()` turns that into particle counts scaled by how far PM2.5 sits over the active line; `densityScale()` tunes count by screen size. This is the "modeled, not measured" core — keep it honest. |
| `lib/nowcast.js` | EPA NowCast for the headline AQI (12-hr weighted PM2.5 → AQI via 2024 breakpoints). |
| `lib/useAsync.js` | The async hook (above). |
| `lib/embedHeight.js` | Posts content height to a parent frame for iframe auto-resize. No-op when unframed; never reads from parent. |

### View layer (`src/viz/`)

- `P5Sketch.jsx` — wraps a p5 sketch so **React owns the DOM node, p5 owns the
  canvas**. It `import()`s p5 lazily (p5 is ~1MB) and calls `instance.remove()`
  on unmount/prop change. **It remounts whenever the `data` prop changes by
  reference** → callers must `useMemo` the data object (AirPage does: `sketchData`,
  `ringsWho`, `ringsLegal`).
- `airParticleSketch.js` — the sketch, `(p, data) => {…}`, **instance mode only**
  (global mode breaks with multiple embeds and React remounts). `data =
  { current, view, mode, hidden }`. Three builders:
  - `buildBaseline()` — Earth's clean atmosphere (shown at `/`, before a search).
  - `buildSource()` — the scattered speck field ("by source"). `Speck` drifts
    with Brownian jitter; hidden source keys and ultrafine are skipped.
  - `buildRings()` — one concentric ring per pollutant ("by pollutant").
    Radii/band/pulse/dot are **canvas-relative** so rings stay inside small
    canvases; exceedance drives ring *density*, not radius.
  - Perf: `pixelDensity(1)` + `frameRate(30)` in `setup()` (big fill-rate win on
    retina; the fields only drift gently).

### UI layer (`src/pages/AirPage.jsx`, `src/components/`)

`AirPage.jsx` is where almost all the product lives. Key pieces:
- `Controls` — the "View" (by source / by pollutant) and "Measured against"
  (legal / WHO) segmented toggles. "Measured against" only shows in the source
  view; the pollutant view renders both lines instead.
- `AqiMeter` — the 0–500 EPA scale as a colored bar (Good→Hazardous, incl.
  maroon) with a ▼ caret at the reading.
- `SourceLegend` — "what's in this breath": % of the modeled breath + raw counts,
  rows tappable to hide (drives the canvas `hidden` set), 0% rows filtered out,
  ultrafine broken out separately, plus a duplicated source line.
- `PollutantList` / `MiniLine` — each pollutant as **two** bars (WHO + US legal)
  showing how many × over each line it sits.
- `ScenarioBar` — the preset buttons; each navigates to `/:id`.
- `RingPanel` — one labeled ring canvas; two stack vertically inside a `bg-cream`
  wrapper so they read as a single split canvas.

Presentation components:
- `Layout.jsx` — header / nav / footer (footer attribution is required).
- `GourmetMediaContainer.jsx` + the `gourmetMediaContainer` CSS in `index.css` —
  the shared "chart card" convention (ink border, hard offset shadow, inset
  graph band, centered source line). The card owns the title/source.
- `LookupInput.jsx` (search form), `Status.jsx` (`Loading` / `ErrorState` — a
  failed lookup must render a styled message, never a blank screen).

### Content & brand
- `src/site.config.js` — **the one content file**: title, tagline, nav,
  attribution, `dataStatus: 'live'`. Edit here, not in components.
- `tailwind.config.js` — the brand palette (`cream #F7F0EF`, `ink #383838`, …)
  and fonts. `cream` intentionally equals the sketch's canvas background.

---

## Gotchas (things that already bit us)

- **`gourmetMediaContainer` CSS beats Tailwind on buttons.** `.buttonsDiv button`
  in `index.css` has higher specificity than utility classes, so the active
  segmented button needs `!bg-black !text-cream` (important modifier) to win.
- **Ring geometry must stay canvas-relative.** The rings were originally sized in
  fixed pixels for an 800px canvas and spilled off the small stacked canvases.
  Anything ring-related should scale to `p.width`.
- **Preview/eval quirk:** in this environment the browser JS console reports
  `window.innerHeight` and `offsetWidth` as **0**, so `vh`/`%` measurements read
  as collapsed and JS-driven scroll/measurement is unreliable. Consequences: the
  readout cap uses a fixed `max-h-[560px]` (not `vh`), and you should **verify
  layout with screenshots**, not `getComputedStyle`.
- **Memoize P5Sketch `data`.** A new object every render remounts the sketch and
  restarts the animation.

---

## Run & verify

```sh
npm install
npm run dev     # http://localhost:5173
npm run build   # production build (Vercel-ready; vercel.json rewrites SPA routes)
```

Try: `/Detroit`, `/90001`, `/cigarette`, `/wildfire`, `/dust-storm`. The dev
server may already be running on 5173.

**Verification workflow that fits the priorities:** make the change → open the
preview → drive it (search a place, toggle views, hide a source, resize to
mobile) → confirm no console errors and nothing overflows → screenshot the
result. The owner reacts to screenshots, so lead with one.

## Recent design intent (so you don't undo it)

- By-pollutant shows **both** lines (no legal/WHO toggle there); the two ring
  canvases are **stacked vertically** in one cream panel.
- Source counts were **reduced** for the heavy-combustion scenarios; don't crank
  them back up.
- The "back to baseline atmosphere" link was intentionally removed from the
  readout (baseline is still reachable via `/`).
- 0% source rows are intentionally hidden.

---
> Source: [juweek/air-diagram](https://github.com/juweek/air-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
