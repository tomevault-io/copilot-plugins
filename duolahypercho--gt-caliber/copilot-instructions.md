## gt-caliber

> You are contributing to an original open-source open-world game built on

# AGENTS.md — repo contract for AI agents

You are contributing to an original open-source open-world game built on
Godot 4.6 (GDScript) with optional C++ GDExtension modules. Humans and agents
follow the same rules; this file is the machine-readable summary.

## Ground truth

- **Task source:** `docs/ROADMAP.md`. Work top-down within the current
  milestone; an unchecked box is a task. Check the box in the same PR that
  completes it.
- **Quality bar:** `docs/VISION.md`. **Layering rules:** `docs/ARCHITECTURE.md`.
- **Never** add assets, code, names, or map layouts from Grand Theft Auto or
  any commercial game. Asset rules: `docs/ASSETS.md` — every new binary asset
  needs a provenance ledger row in the same change.

## Definition of done (every change)

```bash
tools/check.sh   # format + lint + headless import + smoke test + unit tests
```

Must exit 0. If you cannot run Godot headless in your environment, say so
explicitly in the PR body — do not claim checks passed.

## Hard rules

1. `main` stays playable: `godot --path game` must boot to a controllable
   character after your change.
2. Typed GDScript only; tabs; `snake_case` files; `PascalCase` class names.
3. Extract testable logic into plain `RefCounted`/static-function classes
   (pattern: `game/scripts/player/player_motion.gd`) and add a
   `game/tests/unit/test_*.gd` for it.
4. Do not edit `game/addons/**` (vendored), generated `.uid` / `.import`
   files by hand, or `game/bin/**` (build artifacts).
5. No new autoloads, no new third-party addons, no engine-core patches
   without an issue approved by a maintainer first.
6. C++ in `engine/` only with profile evidence per `docs/ARCHITECTURE.md`.
7. One concern per PR. Reference the roadmap line or issue it closes.
8. Scene files (`.tscn`) are hand-editable text — keep diffs minimal and
   never reformat sections you didn't change.

## Scene/scripting conventions (quick reference)

- Signals up, calls down. World scenes are self-contained (streaming-ready):
  no cross-scene node paths; use groups (`player`, `world`) and signals.
- Player spawns are `Marker3D` nodes in the `spawn_points` group.
- Input actions defined in `project.godot`: `move_left/right/forward/back`,
  `jump`, `sprint` (+ built-in `ui_cancel` for mouse release).

## Commit/PR format

- Imperative title ≤ 72 chars, body explains *why*.
- PR template checklist is mandatory; fill it honestly.
- If a change is reverted by CI failure, fix forward only with a root cause;
  otherwise revert cleanly.

---
> Source: [duolahypercho/GT-caliber](https://github.com/duolahypercho/GT-caliber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
