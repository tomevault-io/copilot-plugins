## riso-windowseat

> Procedural risograph films and still art. Each work is one self-contained `index.html` that draws

# riso-windowseat

Procedural risograph films and still art. Each work is one self-contained `index.html` that draws
every pixel in Canvas 2D: no libraries, fonts, images or network calls in the deliverable
(the one exception, `films/window-seat`'s embedded piano recordings, is attributed data inside the
file). External references may be viewed for research, never embedded.

Skills: `riso-film` for animation, `riso-still` for images, `riso-score` for sound. Before
drawing, read `.claude/rules/riso-plates.md` and `docs/visual-development.md`. Technical checks
do not certify artistic quality; say what was and was not looked at or heard.

## Setup

```
cd tools && npm install && npm run setup && npm test   # browsers via playwright-core; ffmpeg-static
```

## Commands (run in `tools/`)

```
node new-riso.mjs --kind film --duration 60 --out ../films/<name>/index.html
node verify.mjs ../films/<name>/index.html
node review.mjs ../films/<name>/index.html
node shoot.mjs ../films/<name>/index.html --range 6.6:6.9:0.0333333333 --sheet
node still.mjs ../prints/<name>/index.html --at 0 --out ../out/<name>.png
node render.mjs ../films/<name>/index.html --fps 30 --size 1080 --engine firefox
node audio.mjs ../films/<name>/index.html --twice --marks 6,13.9
```

Read every sheet you generate. `verify` exits nonzero on failure. Range renders are silent; full
exports mux `renderAudio()`. Firefox is the primary engine. `out/` is disposable and git-ignored.

## Invariants

- Expose `window.__riso = { duration, ready, seek(t), renderAudio?, marks?, shots? }`; every tool
  drives it.
- `seek(t)` is pure in time, independent of seek history. Never call `Math.random()` in a render
  path; stable keys through `rngFor(key)` prevent texture crawl. A hook flags violations in
  `films/`.
- `renderAudio()` is exactly `duration` long at 48 kHz stereo and seeded like the picture;
  `--twice` checks it.
- Bake scenes at displayed size. Never resize a screened bitmap: it causes moiré. Film backing
  stores are 1080, independent of DPR; screen pitch is in device pixels. Larger stills rerasterize
  geometry and screens at the native size.
- No pure black ink. Darks are overprints; keep full knockouts for bright subjects.
- Changing a `Score({ key })` or any `rngFor` key reseeds that output; treat keys as frozen once
  a work is approved.

## Map

| Path | Holds |
|---|---|
| `films/window-seat/` | Showcase film: source, `FILM.md`, sample credits, piano-bank rebuild. |
| `films/lumen/`, `films/emergence/`, `prints/workings/` | Finished examples; workings donates the print kit. |
| `docs/` | Brief, visual development, drawing, scene space, motion, sound, quality bar. |
| `studies/` | Craft, composition, scene-space and sound kits with A/B studies. |
| `.claude/skills/*/examples.md` | Where each technique lives in the shipped sources. |
| `tools/` | Scaffolding, inspection and export harness; `README.md` documents each tool. |

---
> Source: [sevenevesai/riso-windowseat](https://github.com/sevenevesai/riso-windowseat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
