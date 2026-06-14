## velo

> Qt6/FFmpeg non-linear video editor. See README.md for features/architecture.

# Velo — development notes

Qt6/FFmpeg non-linear video editor. See README.md for features/architecture.

## Build & test
- `cmake -B build -G Ninja && ninja -C build` → `build/velo`
- Engine smoke test: `QT_QPA_PLATFORM=offscreen ./build/velo --selftest`
  (must print `selftest: OK`). Run it after engine/model changes.
- GUI check on this machine: X `:0` has **no window manager**; root-window
  grabs are black. Capture the app window directly:
  `WID=$(DISPLAY=:0 xdotool search --name "Velo" | head -1)`
  (the window title ends with the version, e.g. "Untitled — Velo 0.0.1")
  `ffmpeg -f x11grab -window_id $WID -i :0 -frames:v 1 shot.png`
  Drive input with `xdotool mousemove --window $WID <x> <y> click 1`
  (coords are window-relative, 1680×960 default size).

## Conventions
- Model mutations: call `Document::beginUndoStep()` **before** mutating,
  `notifySequenceChanged(seqId)` after. Hold `Document::mutex()` for any
  mutation that isn't already inside a Document method (render/audio/export
  threads read the model under that mutex).
- After any `Track::overwriteInsert`, call `Document::fixupClipIds(seq)` —
  splits create clips with `id == 0`.
- Keyframe times are clip-local seconds (track volume: sequence seconds).
- Sequence ids starting with `__` are internal (source monitor); they are
  skipped by serialization and the media bin.
- New effects: append a `registerEffect({...})` in `src/effects/Effects.cpp`;
  UI/serialization/keyframing pick them up automatically.
- Linked A/V clips share a nonzero `Clip::linkId`; splitting remaps the
  right-hand halves to a fresh linkId.
- Version lives in `CMakeLists.txt` (`project(Velo VERSION x.y.z)`) only;
  releases are cut by pushing the matching `v<x.y.z>` tag (CI builds
  Linux/Windows/macOS bundles and publishes the GitHub release).

---
> Source: [notune/velo](https://github.com/notune/velo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
