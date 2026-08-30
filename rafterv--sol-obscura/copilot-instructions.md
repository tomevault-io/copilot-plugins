## sol-obscura

> description: Sol Obscura OBS widget project — full context for AI code assistance

# Sol Obscura — Cursor Rules
# Place this file at .cursor/rules/sol-obscura.mdc in the project root.
# This gives Cursor full context for the Sol Obscura OBS widget codebase.

---
description: Sol Obscura OBS widget project — full context for AI code assistance
globs:
  - "**/*.html"
  - "**/*.css"
  - "**/*.js"
  - "**/*.md"
alwaysApply: true
---

## Project Overview

Sol Obscura is a Burning Man-adjacent music and art event centred on the **total solar eclipse
of 12 August 2026** in Tortosa, Catalonia, Spain.

This repository contains **OBS Browser Source widgets** — self-contained HTML files that run
inside OBS Studio as live overlays on a projected screen at the venue. They are information art
installations, not broadcast graphics. The guiding philosophy is restraint: the screen should
serve curiosity without competing with the sky.

### Key Event Facts

| Property             | Value                                              |
|----------------------|----------------------------------------------------|
| Totality start       | 20:29:51 CEST (18:29:51 UTC)                       |
| Totality duration    | 1 minute 31 seconds (91 seconds)                   |
| Sun altitude         | 4.7° above western horizon                         |
| Venue                | South of Tortosa, Ebro Delta, Catalonia, Spain     |
| Saros cycle          | 126                                                |
| Audience             | General Catalan / Spanish public, all ages         |
| Totality UTC epoch   | 2026-08-12T18:29:51Z                               |

---

## File Structure

```
./
  countdown-widget.html       # Countdown to totality — OBS top-right corner overlay
  NOAA-solar-wind-overlay.html # Live solar wind data — OBS bottom-third panel
  SDO.html                    # Auto-refreshing NASA SDO solar image — OBS corner widget
  weather-solar.html          # Live solar irradiance panel (0x15)
  weather-uv.html             # Live UV index panel (0x17)
  weather-wind-speed.html     # Live wind speed panel (0x0B)
  weather-wind-direction.html # Live wind direction panel (0x0A)
  weather-pressure.html       # Live relative pressure panel (wh25.rel)
  weather-solar-graph.html    # Live solar irradiance graph panel
  weather-banner.html         # Right-edge combined weather banner
  [future widgets here]
docs/
  sol-obscura-brand-reference.md
  Sol-Obscura-Science-Dashboard-Design-Guide.pdf
  TextandBannerBestPractices.md
.cursor/
  rules/
    sol-obscura.mdc           # This file
```

---

## OBS Scene Architecture

Widgets are loaded as **Browser Sources** in OBS Studio at 1920×1080.
Five scenes run on a timed schedule via OBS Advanced Scene Switcher:

| Scene                    | Time (CEST)         | Widgets active                                      |
|--------------------------|---------------------|-----------------------------------------------------|
| AMBIENT_COSMOS           | Aug 10–12 daytime   | countdown (top-right, small), NOAA overlay, SDO     |
| ECLIPSE_BUILD            | Aug 12 ~18:30–20:25 | countdown (centre, large), NOAA overlay             |
| TOTALITY                 | Aug 12 20:28–20:32  | Minimal text only — NO countdown, NO data overlays  |
| POST_ECLIPSE_REFLECTION  | Aug 12 20:32+       | NOAA K-index, static title card                     |
| NIGHT_AMBIENT            | Nightly 22:00–07:00 | ISS feed, science fact rotator                      |

**Scene 3 (TOTALITY) is the most important design constraint**: the screen steps back.
No urgency, no flashing, no countdown. One line of bilingual text, one SDO image, silence.

---

## Design Principles

These values must be reflected in all code, copy, and layout decisions:

- **Immediacy** — widgets invite presence; they never dominate or mediate the eclipse itself
- **Radical Inclusion** — all text is bilingual (Catalan first, Spanish second); font sizes
  readable from 4 metres; accessible contrast ratios throughout
- **Leave No Trace** — no persistent storage, no cookies, no tracking; widgets are stateless
- **Radical Respect** — no auto-playing audio; no flashing or strobing effects
- **Gifting** — all data sources are free and public; no paywalled APIs

---

## Colour Tokens

All widgets **must** use these CSS custom properties. Do not introduce ad-hoc hex values.

```css
:root {
  /* Core palette */
  --color-bg:           #0A1A1F;   /* Deep Void — primary background */
  --color-solar-gold:   #FFB81F;   /* Solar Gold — primary accent, countdown digits */
  --color-pale-gold:    #FDDC91;   /* Pale Gold — secondary labels, body text on dark */
  --color-text:         #FCF7F7;   /* Warm Off-White — reading text */
  --color-divider:      #656343;   /* Olive Shadow — borders and <hr> ONLY, never text */
  --color-grey-mid:     #A9A9A9;   /* Mid Grey — disabled / placeholder text */
  --color-grey-light:   #C1C1C1;   /* Light Grey — borders */

  /* Extended dashboard tokens */
  --color-corona:       #E05C00;   /* Corona Orange — alerts on deep-void bg only */
  --color-corona-bright:#FF7A2F;   /* Bright Corona — alerts on teal panel surfaces */
  --color-panel:        #1A4A4F;   /* Twilight Teal — panel/card surfaces */
  --color-iss-blue:     #C8D8E8;   /* ISS Blue — Earth feed and geomagnetic data */
  --color-void:         #050508;   /* Totality Black — Scene 3 background only */
}
```

### Colour Rules (enforced)

1. `--color-divider` is **only** for `border`, `border-top`, and `<hr>` elements.
   Never use it as a `color` property on text — it fails WCAG AA contrast on all backgrounds.
2. `--color-corona` (#E05C00) passes contrast on `--color-bg` (ratio 4.83) but **fails** on
   `--color-panel` (ratio 2.67). Use `--color-corona-bright` (#FF7A2F, ratio 3.78) for any
   alert text rendered on teal panel backgrounds.
3. All text must meet **WCAG AA**: ≥4.5:1 for normal text, ≥3:1 for large text (≥24px or
   ≥18.67px bold). The primary palette achieves 5.68–16.76 on standard backgrounds.
4. Opacity modifiers on text are acceptable if the **blended** colour still passes — pale-gold
   at 0.65 opacity on deep-void bg blends to ~#A79869 (ratio 6.22 ✅).

---

## Typography

```css
:root {
  --font-display: 'Urbanist', sans-serif;      /* Headings, event title, spaced caps */
  --font-body:    'Poppins', sans-serif;        /* Labels, body copy, bilingual text */
  --font-data:    'Space Grotesk', sans-serif;  /* All numeric data values */
}
```

Always include this Google Fonts import in every widget `<head>`:

```html
<link href="https://fonts.googleapis.com/css2?family=Urbanist:wght@300;400;600&family=Poppins:wght@300;400&family=Space+Grotesk:wght@400;500&display=swap" rel="stylesheet">
```

### Type Scale for OBS 1080p

| Role                 | Font          | Weight | Size   |
|----------------------|---------------|--------|--------|
| Primary data value   | Space Grotesk | 500    | 96px   |
| Countdown digits     | Space Grotesk | 500    | 52px+  |
| Section heading      | Urbanist      | 400    | 40px   |
| Data labels          | Poppins       | 300    | 18px   |
| Secondary labels     | Poppins       | 300    | 12–16px|

Minimum font size for any visible text: **10px** (used only for very secondary chrome).
Prefer larger — text must be readable from 4 metres on a projected screen.

---

## Bilingual Text Pattern

All user-facing labels cycle through three languages in sequence using **CSS animation only** — no JavaScript, no i18n frameworks. Order is always: **Catalan → Spanish → English**.

Each language phrase is a separate `<span>` stacked in the same position using `position: absolute` with staggered `animation-delay`. The data value itself does not animate — only the label rotates.

### Rules

- Cycle timing: **10 seconds per language**, 30 second total loop — long enough to read comfortably at a meditative pace
- Use a **gentle fade** (not a slide or flip) — consistent with the meditative pacing of the installation
- The data value (number + unit) is **never animated** — only the label rotates
- Set `min-width` and `height` on the cycling container to prevent layout shift as absolute-positioned spans swap
- Every widget must show all three language spans in the DOM even when visually hidden via opacity — do not inject them with JavaScript
- `aria-live="polite"` on the cycling container is acceptable but not required for OBS use
- For **Scene 3 (TOTALITY)** the bilingual text is static — do not apply lang-cycle animation to the totality text; it is a single calm statement, not a label

---

---

## Live Data Sources

NOAA endpoints are free and public (no API key). Local weather data comes from the venue gateway.

| Data                    | URL                                                                          |
|-------------------------|------------------------------------------------------------------------------|
| Solar wind speed/density| https://services.swpc.noaa.gov/products/solar-wind/plasma-1-day.json        |
| Solar wind Bz/Bt field  | https://services.swpc.noaa.gov/products/summary/solar-wind-mag-field.json   |
| Geomagnetic K-index     | https://services.swpc.noaa.gov/products/noaa-scales.json                    |
| Aurora forecast         | https://services.swpc.noaa.gov/json/ovation_aurora_latest.js                |
| Local weather live feed | http://192.168.4.48/get_livedata_info                                       |
| Local weather proxy     | http://127.0.0.1:8787/weather                                               |
| SDO solar imagery       | https://sdo.gsfc.nasa.gov/assets/img/browse/                                |
| ISS live feed           | https://www.youtube.com/watch?v=sWasdbDVNvc                                 |
| Eclipse live stream     | https://www.youtube.com/watch?v=DtjbfQtqlsY                                 |
| Eclipse tracker         | https://www.timeanddate.com/eclipse/solar/2026-august-12                    |

### Fallback Strategy

Widgets must degrade gracefully. Every fetch should have a try/catch that:
1. Displays a static fallback value or "—" placeholder
2. Shows a non-alarming status message (e.g. "Dades no disponibles / Datos no disponibles")
3. Retries silently on the normal refresh interval — no error modals, no broken layouts

---

## OBS Browser Source Behaviour

- `background: transparent` on `body` for overlay widgets (countdown, SDO)
- `background: var(--color-bg)` on `body` for full-panel widgets (NOAA)
- Do **not** use `position: fixed` — OBS Browser Sources don't scroll
- Disable `overflow: hidden` only intentionally — stray content causes black bars in OBS
- Set `width: 100vw; height: 100vh` — OBS sizes the browser to the source dimensions
- Animations: use `transition` and `CSS keyframes`, not JS `setInterval` for visual effects
- Refresh intervals: data fetches every 60s; SDO image every 60s; countdown ticks every 1s
- Never auto-play audio

---

## Totality Moment — Critical Constraints

The period **20:28–20:32 CEST** (Scene 3) has hard rules:

- No countdown timer visible
- No live data numbers — they are a distraction
- Background: `--color-void` (#050508)
- Text: one bilingual line maximum — `"Totalitat: 1 minut 31 segons — 20:29:51"`
- One static SDO coronal image, no animation
- No pulsing, blinking, or colour transitions
- The JS epoch for totality is `new Date(Date.UTC(2026, 7, 12, 18, 29, 51))`

---

## Code Style

- Vanilla HTML/CSS/JS only — no frameworks, no bundlers, no npm
- Each widget is a **single self-contained `.html` file** with inline `<style>` and `<script>`
- Use CSS custom properties (`var(--color-*)`) — never hardcode hex values in component styles
- `const` and `let` only — no `var`
- Use `async/await` with `try/catch` for all `fetch()` calls
- Add a brief HTML comment at the top of each file:
  ```html
  <!-- Sol Obscura · [Widget Name] · OBS Browser Source · 1920×1080 -->
  ```
- No external JS libraries (no jQuery, no lodash, no chart libraries)
- Keep each widget under ~300 lines — split into multiple widgets rather than one complex one

---

## Known Contrast Issues (Fixed in v2+)

Three issues were identified during a contrast audit. Any new work should apply these fixes
and must not reintroduce the original patterns:

| Issue | Original | Fix |
|-------|----------|-----|
| `.totality-time` / `.sublabel` text | `color: var(--color-divider)` (ratio 2.90) | Use `var(--color-pale-gold)` at 0.65 opacity |
| `.timestamp` on teal panel | `color: var(--color-divider)` (ratio 1.60) | Use `var(--color-pale-gold)` at 0.65 opacity |
| `.value.alert` on teal panel | `color: var(--color-corona)` (ratio 2.67) | Use `var(--color-corona-bright)` (#FF7A2F, ratio 3.78) |

---
> Source: [rafterv/sol-obscura](https://github.com/rafterv/sol-obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
