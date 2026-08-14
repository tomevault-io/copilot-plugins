## metro-gnome-v3

> This is the final build spec, ready to hand to an agentic coding tool. All open decisions from planning are resolved (see status log at the bottom). Treat this as the source of truth for the rebuild.

# metro-gnome — build spec

This is the final build spec, ready to hand to an agentic coding tool. All open decisions from planning are resolved (see status log at the bottom). Treat this as the source of truth for the rebuild.

---

## 1. Project overview

A rebuild of an existing metronome site into four tools sharing one accurate audio engine:

- `/metronome` — metronome with per-beat customization
- `/strum` — strum machine with a hybrid sample/synthesis guitar engine and a song library
- `/tuner` — microphone-based tuner with a custom range
- `/timing-game` — tap-along rhythm game (ported from the existing site, lowest priority)

**Stack:** React + Vite.

**Repo history:** two earlier attempts exist (`metro-gnome` on GitHub Pages, and an unfinished `Metro-gnomeV2` React rewrite). Both drove timing with `setInterval`, which is not sample-accurate and drifts over time — this is being replaced (see below), not carried forward.

---

## 2. Shared audio engine (build this first)

Everything else depends on this. Replace `setInterval`-driven timing with a look-ahead scheduler:

- A coarse `setInterval` (~25ms) checks whether anything needs to be queued in the next ~100ms window
- Each event's actual playback time is computed as an exact `audioContext.currentTime` offset, not "whenever the timer fires"
- Any visual element (beat flash, swinging indicator, strum playhead) reads timestamps from this same schedule via `requestAnimationFrame`, rather than running its own independent timer or CSS animation

This one module underlies the metronome, the strum machine, and the timing game. The tuner does not use it (it consumes mic input rather than driving playback), but should live in the same shared audio utilities file for consistency.

---

## 3. Metronome page (`/metronome`)

**Core controls**
- BPM range 20–300: tap-tempo button, +/- steppers, slider, and direct numeric input, all kept in sync
- Time signature selector, presets only: 2/4, 3/4, 4/4, 5/4, 6/8, 7/8, 9/8, 12/8 — no custom numerator/denominator input

**Visual indicator**
- One metronome graphic centered at the top of the page (swinging needle/pendulum look), driven directly by the shared scheduler
- The row of beat cells (below) also flashes its own corresponding cell as each beat plays — top-level swing and per-beat detail visible at the same time

**Per-beat customization**
- A row of cells, one per beat in the measure. Each cell is independently set to **accent**, **normal**, or **muted/silent**, and assigned a sound from the sound palette
- The row automatically resizes to match the time signature (3/4 → 3 cells, 6/8 → 6 cells, etc.), with sensible accent defaults on beat 1 (and beat 4 for compound meters like 6/8)
- Sound palette: reuse the existing `click_1/2/3.wav` samples, plus a few generated with simple Web Audio oscillators (woodblock, cowbell, digital beep) rather than more recorded assets
- `guitarLoop.wav` from the old site is not carried forward — drop it

---

## 4. Strum machine page (`/strum`)

**Core structure**
- A chord progression: an ordered list of chords, each with a duration in beats or measures
- A strum pattern editor: a grid aligned to the time signature, each subdivision slot set to down-strum, up-strum, mute/chuck, rest, or accent
- Song sections (intro/verse/chorus/etc.), each with its own progression and pattern
- Tempo, key, and capo controls — capo/key changes transpose both displayed chord names and audio
- Swing/shuffle amount, and per-strum dynamics (accent vs ghost strum)
- Shares its tempo/clock with the metronome page's scheduler

**Rhythm pattern library**
Preset patterns, selectable per section and editable after picking:
- **Boom-chuck** — alternating bass note on beats 1 and 3 (root, then a second bass note such as the fifth or next-lowest available string), chord chuck on beats 2 and 4
- **Drone** — a genuinely sustained, bowed-string-like tone (not a decaying pluck) for a held, violin-ish character
- Standard folk/pop strum patterns (e.g. D-DU-UDU), a waltz pattern, a reggae skank, and a couple of fingerstyle-adjacent arpeggiated patterns
- Custom: build and save a pattern from the grid

**Guitar sound engine — hybrid approach**
1. **Sample layer:** a set of recorded strums covering open-position chords in the most common keys. Loaded via Vite's `import.meta.glob('/src/samples/**/*.wav')`, so any file dropped into `src/samples/` following the naming convention `{root}-{quality}_{position}_{strumDirection}.wav` (e.g. `E-major_open_down.wav`) is automatically discovered at build time — no manifest file, no registration step.
2. **Synthesis fallback:** Karplus-Strong string modeling for anything outside sample coverage. Each of the 6 strings is an independent plucked-string oscillator, triggered per note in the chord voicing with a small stagger between strings (5–15ms) to simulate a real strum roll. A working reference implementation is provided in `guitar-synth-demo.html` alongside this spec — use its `pluckBuffer`/`playString`/`strum` logic as the starting point for the real engine, and its `droneVoice` (filtered sawtooth, slow attack, vibrato) for the drone pattern specifically, since Karplus-Strong cannot sustain the way a bowed tone needs to.
3. **Lookup logic:** on chord/strum-direction request, check the sample lookup table first; if no match, fall back to the synth engine automatically.

**Song / backing track library**
Songs are data, not audio: each is a small JSON file (key, tempo, time signature, chord progression with timing, strum pattern per section) rendered live through the engine above. A `src/songs/` folder, discovered the same way via `import.meta.glob`, gives a searchable catalog with no manual index to maintain.

- **Phase 1 (this build):** a catalog of bluegrass, folk, and traditional Irish tunes using only basic open chords
- **Phase 2 (later, not in this build):** expand the chord parser to handle sevenths, slash chords, and extended/altered voicings, to support jazz standards and anything outside simple open-chord repertoire

---

## 5. Tuner page (`/tuner`)

Mic input via `getUserMedia`, pitch detection via autocorrelation or YIN against an `AnalyserNode`/`AudioWorklet` buffer. Requires HTTPS (GitHub Pages provides this) and a one-time mic permission prompt.

**Range:** a single semitone-offset control from −4 to +2 (default 0 = E standard), applied uniformly across all six strings.

| String | E standard | Low bound (−4 semitones) | High bound (+2 semitones) |
|---|---|---|---|
| 6 | E2 (82.41 Hz) | C2 | F♯2 |
| 5 | A2 (110.00 Hz) | F2 | B2 |
| 4 | D3 (146.83 Hz) | A♯2/B♭2 | E3 |
| 3 | G3 (196.00 Hz) | D♯3/E♭3 | A3 |
| 2 | B3 (246.94 Hz) | G3 | C♯4/D♭4 |
| 1 | E4 (329.63 Hz) | C4 | F♯4 |

**UI:** offset slider/stepper with note names relabeling live, a needle or strobe-style indicator, cents readout, nearest-string auto-detect, and a brief "hold steady" confirmation before flashing green (avoids flicker on a wavering pluck).

---

## 6. Timing game (`/timing-game`)

Port the existing tap-along feedback feature as-is: tap along, get feedback on whether the tap landed within tolerance, red/green flash. The only change is reading timing off the shared scheduler's real audio timestamps instead of `Date.now()`-adjacent values. Lowest priority page — build last.

The old site's `testing/` folder (MIDI input testing, boombox tutorial, other experiments) is not carried into this rebuild.

---

## 7. Visual style

**Palette** (use as CSS custom properties, not hardcoded hex scattered through components):
- `--bg`: `#14181A` — page background
- `--surface`: `#1E2426` — panel/card surface
- `--teal`: `#2E7D78` — structural accents, borders, secondary actions
- `--red`: `#C9432F` — primary accent: active/on-beat states, primary buttons
- `--cream`: `#ECE6DA` — body text, pixel-art highlight tones
- `--gold`: `#C9A227` — rare emphasis only (e.g. an "in tune" or "on beat" flash) — not a workhorse color

**Typography:** a plain, clean sans for all functional UI (BPM numbers, labels, buttons, form controls). Do not use a pixel/8-bit font for body text or controls — reserve the pixel-art character entirely for the mascot illustrations and any logo/wordmark treatment. The contrast between playful mascot art and quiet functional UI is intentional.

**Rendering style for all illustration assets:** pixel art, visible pixel grid, thick black outlines, flat color fills, no gradients or anti-aliasing — matching the reference images below.

---

## 8. Assets

Five final pixel-art images are provided alongside this spec, generated to match the palette above:

| File | Use |
|---|---|
| `favicon-gnome.png` | Site favicon — the gnome's hat with musical notes |
| `sitting-gnome.png` | Homepage / hero image — gnome sitting cross-legged with a guitar |
| `metro-gnome.png` | Metronome page illustration — gnome beside a pendulum metronome |
| `strumming-gnome.png` | Strum machine page illustration — close-up of gnome strumming, chord diagrams in background |
| `tuner-gnome.png` | Tuner page illustration — gnome with a hand cupped to his ear, sound wave from a guitar headstock |

Place these in `public/images/` and reference them directly by path (Vite serves `public/` at the site root). The favicon should additionally be set up as `favicon.ico`/`favicon.png` referenced from `index.html`'s `<head>`, resized as needed for browser tab display (test at actual 16–32px size — see notes in the image-prompt file about favicons losing legibility when shrunk).

---

## 9. Suggested build order

Build in dependency order, not spec order:

1. **Shared scheduler module** — the foundation everything else needs
2. **Metronome page** — simplest page, validates the scheduler and visual sync together
3. **Tuner page** — fully independent, can be built and tested in isolation
4. **Strum machine** — most complex; benefits from a proven scheduler and chord-shape system already in place
5. **Timing game** — lowest priority, small effort once the scheduler exists

---

## 10. Status log

- Stack: React + Vite — confirmed
- Guitar sound: hybrid sample + Karplus-Strong synthesis, with `guitar-synth-demo.html` as the synthesis reference implementation — confirmed
- Metronome: preset time signatures only, auto-sizing beat-cell row, top-center swinging indicator + per-cell flash — confirmed
- Tuner: −4 to +2 semitone range (C standard to F♯ standard) — confirmed
- Timing game: kept, own page, lowest priority — confirmed
- Old `testing/` folder: dropped — confirmed
- Visual style: dark palette + pixel-art assets — confirmed, final images attached

---
> Source: [santimeza/metro-gnome-v3](https://github.com/santimeza/metro-gnome-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
