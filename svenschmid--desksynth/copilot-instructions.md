## desksynth

> DeskSynth is a pixel pocket synth that lives on your desktop. Electron, Web

# AGENTS.md

DeskSynth is a pixel pocket synth that lives on your desktop. Electron, Web
Audio, and one hand-drawn pixel buffer. Read `docs/README.md` before changing
anything substantial.

## Run

```bash
npm start                          # live window
node tools/shots.mjs mood:ecstatic # offscreen render to .shots/, for pixel work
```

## Rules

1. **Documentation is part of the change.** If you change a system, update its
   document in the same commit. Each doc lists the files it covers at the top.

   | You touched | Update |
   | --- | --- |
   | `main.js`, `preload.js`, `src/app.js`, `src/core/layout.js` | `docs/architecture.md` |
   | `src/audio/engine.js`, `presets.js`, `dsp.js` | `docs/audio.md` |
   | `src/audio/scales.js`, `src/core/keymap.js` | `docs/harmony.md` |
   | `src/core/pet.js`, `src/render/face.js` | `docs/pet.md` |
   | `src/render/*` | `docs/rendering.md` |
   | anything that changes how it looks or feels | `docs/style.md` |
   | tooling, scripts, gotchas | `docs/development.md` |

   New system that fits none of these? Add a document and list it in
   `docs/README.md`.

2. **Follow `docs/style.md`.** It encodes the decisions that keep the thing
   coherent — integer pixels, one accent colour per preset, additive light,
   feedback on every input.

3. **Look at it before you claim it works.** Use the screenshot harness for
   anything visual.

---
> Source: [SvenSchmid/desksynth](https://github.com/SvenSchmid/desksynth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
