## cyninja-prompt-demo

> Godot 4.6 2D pixel-art side-scroller metroidvania prototype. 640x360 viewport,

# CyNinja

Godot 4.6 2D pixel-art side-scroller metroidvania prototype. 640x360 viewport,
integer scaling, nearest-neighbor filtering. Protagonist: a half-robot cyborg
ninja. Currently in the animation-prototyping phase — the deliverables are the
character's clips and a recorded showcase.

The art/animation pipeline is ported from the Cozy Coven project: every asset
is procedural GDScript, no image files are authored by hand.

## Layout

- `art/<category>/<name>.gd` — every asset is a `@tool` GDScript extending
  `PixelAsset`, drawn procedurally in `build(img)`. See `docs/STYLE.md` for
  the hard rules and helper list.
- `art/palette.gd` — shared palette, the "Neon Rain" night ramp.
- `art/{skeleton,paint,motion,compositor,puppet_asset}.gd` — the **Ink &
  Bones** animation system: float-angle bones + IK, parts painted at 4x
  supersample, clips as eased curves, verlet cloth, graded back onto the 1x
  pixel grid. See `docs/ANIMATION.md` — read it before touching a character.
  Shared infrastructure lives at `art/` root, never in a subdirectory: the
  gallery and `audit_edges.gd` scan `art/<category>/` and call `new()` on
  everything they find.
- `puppets/cyninja.gd` — the protagonist, the project's worked example of a
  puppet asset (see `docs/CHARACTER.md`). Puppet characters live in
  `puppets/`, NOT under `art/` (the legacy audit duck-types anything there).
- `art/{pose,track,clip,rig,rigged_asset}.gd` — the RETIRED integer-offset
  animation system, kept for its selftest/testrigs and the reference rig at
  `tools/legacy/cyninja_rigged.gd`. Do not build new characters on it.
- `scenes/` — `showcase` (the recorded demo, and the project's main scene),
  `explainer` (the recorded technique video: bones, curves and the grade drawn
  live over the character, then the same clip re-baked with one field changed),
  `gallery` (asset grid),
  `anim_gallery` (clips playing live, scans both `art/` and `puppets/`).
- `tools/` — `capture.tscn` and `audit_edges.gd`; `puppet_audit` /
  `puppet_selftest` / `puppet_metrics` plus `tools/puppet/` (bake_clips,
  rest_compare) for the puppet system; `anim_sheet` / `anim_audit` /
  `anim_selftest` / `anim_restdiff` / `anim_metrics` for the legacy system;
  `solo` and `solo_batch` for one asset at a time; `render.sh` / `record.sh`
  for stills and video.
- `docs/` — `STYLE.md` (the hard rules and the look), `ANIMATION.md` (the
  puppet system), `CHARACTER.md` (the protagonist's rig and clips).
  `docs/media/` holds the committed renders (GIFs, strips, stills) the README
  embeds — regenerate from `shots/` output rather than editing by hand.
- `shots/` — rendered screenshots and the bake cache, gitignored scratch.

## Rendering

Always go through `tools/render.sh`, never bare `godot`. Concurrent godot
instances clobber the shared `.godot` import cache; the script serializes them
behind a pid-verified lock and enforces a per-render timeout.

```sh
tools/render.sh --path . res://tools/capture.tscn -- scene=... out=...
tools/render.sh --headless --path . -s tools/audit_edges.gd
tools/render.sh --headless --path . -s tools/puppet_audit.gd
tools/render.sh --headless --path . -s tools/puppet_selftest.gd
tools/render.sh --headless --path . -s tools/anim_audit.gd
tools/render.sh --headless --path . -s tools/anim_selftest.gd
```

Those five audits are the quantitative checks. All must be green.

Video goes through `tools/render.sh` too, via `tools/record.sh` — Godot's
Movie Maker mode on a fixed timestep, then ffmpeg to an mp4 sized for social
media. It takes the same lock, so it is safe alongside everything else.

```sh
tools/record.sh --scene res://scenes/showcase.tscn --seconds 15
```

Do not give new shared scripts a `class_name` — it only registers after the
Godot editor rescans, which breaks every headless run until someone opens the
editor. Preload as a constant, or `extends "res://path.gd"`.

---
> Source: [tvanhens/cyninja-prompt-demo](https://github.com/tvanhens/cyninja-prompt-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
