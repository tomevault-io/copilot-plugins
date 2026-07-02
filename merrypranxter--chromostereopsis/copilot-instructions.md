## chromostereopsis

> > Persona carried over from the original Copilot custom agent so any

# Agent guide — chromostereopsis

> Persona carried over from the original Copilot custom agent so any
> coding agent (Copilot, Claude, etc.) picks up the same context.

## Role

Generate pure red/blue fields that create depth illusions without
glasses. This repo is the **naked-eye** cousin of `chromadepth`: no
glasses, no shutters, no headset — the viewer's own eye does the 3D via
longitudinal chromatic aberration. Includes a polarity demo and
breathing layers.

## Before writing code

1. Read [`repo_seed.txt`](./repo_seed.txt) for the full creative brief
   and math references.
2. Read [`context.manifest.json`](./context.manifest.json) for
   dependencies and priority.
3. Check [`docs/visual-targets.md`](./docs/visual-targets.md) for the
   intended aesthetic output, and
   [`docs/perceptual-math.md`](./docs/perceptual-math.md) for the why.
4. Verify shared dependencies are imported, not duplicated.

## Code style

- WebGL2 / GLSL ES 3.00 shaders, clear uniform names, real comments.
- Modular ES6 JS with imports. No global state beyond the single render
  `state` object in `main.js`.
- Reuse shared libraries (`color_systems`, `chromatic_aberration`)
  rather than reimplementing. Where those ecosystem libs aren't vendored
  here, the equivalent helper is inlined and labelled with its origin.
- JSDoc on public JS functions.

## Output expectations

- Runnable code that matches the visual targets.
- Parameter tunables (uniforms, `src/data/presets.json`) for
  interactivity.
- Document gotchas and known limits in comments.
- When in doubt, prefer the mathematically correct implementation over
  the shortcut.

## Non-negotiables (the effect is fragile)

- **Max saturation only.** Pure `#FF0000` / `#0000FF`. Funnel every
  color through `enforceMaxSaturation`. Orange/cyan are too weak.
- **Minimal post.** Heavy finishers kill it. `ca-enhance` stays a
  whisper.
- **It varies per person** — some viewers see it reversed. Say so in UI
  and docs; never claim a single fixed direction.

## Ecosystem

Part of the **Color Lab** batch (lane 3, perceptual). Consumes
`chromatic_aberration` and `color_systems`. Sibling of `chromadepth`
(cross-link hard). Adjacent: `anaglyph_stereo`, `parallax_depth_fields`,
`pulfrich_effect`, `wiggle_stereoscopy`. Keep interfaces clean and
documented for cross-repo reuse.

---
> Source: [merrypranxter/chromostereopsis](https://github.com/merrypranxter/chromostereopsis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
