## claude-motion-studio

> This repo turns an **idea** into a **rendered motion graphic** with Remotion (React → video).

# Claude Motion Studio — orchestration guide

This repo turns an **idea** into a **rendered motion graphic** with Remotion (React → video).
You (Claude) are the designer + engineer: lock the concept, design the look, write the
composition, render it.

## Skills (auto-loaded from `.claude/skills/`)
- **`remotion-motion`** — read it for the build/render workflow, the animation toolkit, the
  asset pipeline, formats, and Remotion gotchas. Covers every style.
- **`motion-design`** — read it for color, type, spacing, elevation, glass, depth, and the
  "make it premium" checklist. Apply it to every screen.

## Process — always idea → style → build → render
1. **Idea** — confirm concept, audience, format (9:16 reel / 4:5 post / 1:1 / 16:9), length, one core message.
2. **Style** — palette + reference + mood (motion-design). Confirm the look before scaffolding.
3. **Build** — add `src/compositions/<Name>.tsx`, compose from `src/lib/primitives.tsx`, register a `<Composition>` in `src/Root.tsx`.
4. **Render** — `npx remotion still <Id> out/x.png --frame=N` to QA, then `npx remotion render <Id> out/x.mp4`.

## Conventions
- Drive all motion off `useCurrentFrame()`; use `interpolate` (precise) + `spring` (organic); always `clamp`.
- Reuse the toolkit (`Reveal`, `MaskUp`, `Scene`, `useCount`, `StatusBar`) — don't re-derive timing.
- Use the `APPLE` / `SMOOTH` easings — never animate linearly.
- Precompute randomness with `mulberry32` at module scope (never `Math.random()` in render).
- Keep user-facing copy out of components when it should be data — pass it via input `--props`.
- Output naming: `out/<id>.mp4` / `out/<id>-<frame>.png`.

## Map
- `src/Root.tsx` — composition registry (render by id).
- `src/lib/primitives.tsx` — easings, Reveal/MaskUp, Scene camera, device chrome, PRNG.
- `src/lib/design.ts` — tokens + example theme (swap per brand).
- `src/compositions/*` — one file per video. `KineticIntro`, `DataReel`, `AppShowcase` are examples.
- `src/generate/*` — gpt-image engine, chroma-key, sfx synth.
- `public/` — fonts, images, audio.

---
> Source: [MoussaabBadla/claude-motion-studio](https://github.com/MoussaabBadla/claude-motion-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
