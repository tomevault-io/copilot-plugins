## batch-tracker-repo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Windows/CUDA VFX matchmove batch tool: raw plate frames in, 2D point tracks for
3D Equalizer out. Four stages chained behind one local NiceGUI app:

1. **Analyze** — Qwen2.5-VL describes each shot (movers, occluders, bad-track regions,
   depth layers, parallax) → deterministic heuristics turn that into SAM3 include/exclude
   prompts and a per-shot track strategy.
2. **Mask** — SAM3 renders per-frame keep/ignore alpha mattes from those prompts.
3. **Track** — SynthEyes (default) or TAPNext++ (GPU fallback), mask-gated.
4. **Export/publish** — classic 3DE 2D-track ASCII, copied into the studio shot tree.

Everything runs offline and locally: weights are loaded with `local_files_only=True` and
will error rather than download.

## Commands

```bat
app\BotBatchTracker.bat                      REM the launcher: NiceGUI app on :8080
```

The bat sets `TMP`/`PIP_CACHE_DIR` inside `runtime/`, exports `BTR_SAM3_WEIGHTS` and the
SynthEyes paths, kills whatever holds port 8080, then runs `app/app_nicegui.py` with
`runtime\python311\python.exe` (or `.venv` if present). **cwd must be the repo root** —
`app/` is both a package and the dir holding `app.py`.

There is no test framework. Verification is done with three scripts, all pure-Python
(no torch/cv2 except `check_per_track`), run from the repo root:

```bat
runtime\python311\python.exe tools\check_per_track.py
REM self-check of the per-track-policy plumbing on synthetic frames (~seconds, no GPU).
REM Catches seed-measurement/track-column misalignment, which otherwise produces a
REM successful run with quietly wrong numbers.

runtime\python311\python.exe tools\eval_refs.py refs\gt4k --bot <export>.txt
runtime\python311\python.exe tools\eval_refs.py refs\gt4k --bot new.txt --baseline refs\gt4k\baseline_bot.txt
REM one row per hand-tracked reference, labelled by feature kind (corner/blob/edge/dense).
REM This is the gate for any tracking-accuracy change: an averaged single-track number
REM hides a rule that helps corners and hurts blobs.
REM NOTE: refs\gt4k holds ONE corner, and the plate it came from is not on this machine --
REM its locked numbers (7.00/4.18/0.94/1.22/1.36) can be re-read but not re-run. Producing a
REM NEW measurement needs bench\ below, or a reference whose footage is present.

runtime\python311\python.exe bench\make_synth.py --out bench\synth\lab02 --frames 100 --hard
runtime\python311\python.exe bench\run_bench.py   --shot bench\synth\lab02 --tag base
runtime\python311\python.exe bench\score_synth.py bench\synth\lab02 --run test --baseline base
REM synthetic shot with EXACT ground truth for every seed, scored per feature class. Covers
REM what one hand-tracked corner cannot; measures localisation only (one plane -> no parallax,
REM no occlusion, and every track on it comes out good). See bench\README.md.

runtime\python311\python.exe -m app.compare_tracks bot.txt manual.txt --bot-track BWD_0010 --ref-track 658
REM one bot track vs one artist track: deviation, first-step error, fast-vs-slow bias,
REM smoothness, worst frames. Use while chasing a single fault.
```

Reference folder layout (`refs/<name>/`): `manual.txt` (hand tracks, 3DE ASCII),
`refs.json` (`{"658": "corner", ...}`), optional `baseline_bot.txt`, `baseline.json`.
Pairing is positional-by-proximity, not by name.

```bat
runtime\python311\python.exe tools\make_lk_reference.py --mp4 D:\Jefrin\IN\SH004.mp4 ^
    --out refs\SH004_lk --quality 0.01 --min-dist 30 --max-closure 1.0 --n 6
REM Builds a reference for a real plate when no hand track exists, using pyramidal
REM Lucas-Kanade -- a different algorithm family from the bot's NCC+ECC refine, so the two
REM do not share pixel-locking bias. Each track is run forward then all the way back; the
REM round-trip CLOSURE bounds that reference's own precision and is recorded per track in
REM reference.json. NOT ground truth: a fault both methods share is invisible to it, so read
REM the closure before believing any row. Folders are named `_lk` to keep the provenance
REM obvious next to real hand tracks.
```

Model weights, the portable interpreter, `out/`, `_batches/`, and `backup/` are
gitignored. `CHECKPOINTS.txt` is the by-hand weight download sheet; `setup_bot.bat`
does it automatically.

## Architecture

### UI / backend split
`app/app.py` (~3.2k lines) is the **entire backend** — loaders, workers, `AppState`,
path helpers — plus a legacy Gradio UI that only builds under `__main__`.
`app/app_nicegui.py` is the live front-end: it loads `app.py` as a file under the module
name **`btr_backend`** (aliased `be`) because the sibling `app/` package would shadow
`import app`. Adding backend behaviour means editing `app.py` and reaching it through
`be.` from the UI.

### Module loading (deliberate, fragile, don't "clean up")
- The embeddable interpreter's `._pth` blocks auto-path insertion, so `_bootstrap_paths()`
  in `app.py` rglobs for `core/io_parsers.py` and injects its parent. The decisioning
  package is imported as `core.*` but **lives at `pipeline/llama/core/`**.
- `_load_tracker_direct` / `_load_syntheyes_direct` / `_load_sam3_direct` / `_load_qwen_robustly`
  try the package import first, then fall back to `spec_from_file_location`. Failures are
  captured into `*_IMPORT_ERROR` strings and surfaced in the log, not raised — a missing
  backend degrades instead of crashing the app.
- Heavy modules are loaded lazily per stage so VRAM staging works: Qwen freed after
  Analyze, SAM3 freed before Track, `_free_vram()` before handing the GPU to SynthEyes.

### State and job model
Single-user, single-session. `AppState` (one shared instance) + `ShotData` per shot hold
everything; `JOB_QUEUE`, `CURRENT_JOB_THREAD`, `STOP_EVENT`, `LAST_PROGRESS*` are module
globals in `app.py`. **Do not open multiple browser tabs** — they share one state.
Workers (`worker_analyze`, `worker_mask`, `worker_track`, `worker_pipeline`) run on one
background thread, push `DONE_*` / `GUIDE_PATH_UPDATE:` sentinels onto `JOB_QUEUE`, and the
UI's 1 s `poll()` drains it. Workers must emit their `DONE_*` on the failure path too, or
the UI stays stuck busy.

Every tuning knob is a field on `AppState` (see `app/app.py:1108`), copied into
`RunnerConfig` (`app/tracker_core.py:27`) at track time. Comments on those fields record
*why* each default is what it is — read them before changing one.

### Folders: local work dir vs studio tree
Shots come from a studio share: `<shows_root>/<show>/<shot>/in/plates/<version>/`.
The pipeline **computes in a local work dir** (`runtime/_work/<show>`, override `BTR_WORK`)
so the cross-shot guide and mask-reuse logic stay intact, then `publish_shot()` copies
finished artifacts to `<show>/<shot>/mid/cmm/bot_tracks/{*_2Dtracks__<backend>.txt,
masks/, analysis/, logs/}`. Per-shot proxies and mp4 renders cache in `<studio>/cache`
so they survive across sessions. Scan reads `has_analysis/has_masks/has_tracks` from the
studio tree, so badges survive a restart and read the same from any workstation.

Work-dir naming the publisher depends on: `<shot>__<task>__<backend>.txt`,
`<shot>__track.log`, `<shot>/masks*/`, `_batches/batch_<ts>/mask_guidance.json`.
Publishing is per shot and per backend, because a batch can fall back mid-run.

### Analyze
`worker_analyze` → Qwen batch over downscaled (1280px) proxies → `core.bridge.build_batch_tracker_json`
merges Qwen signals, client requirements (`requirements` file + UI-typed
`manual_requirements.json`) and `core/heuristics.py` term lists into `mask_guidance.json`.
Prior guides are folded in (`_merge_prior_guide_shots`) so the newest guide always holds
every analyzed shot.

**Decisioning is Qwen-only** (changed 2026-07). `_ensure_ollama` / `_free_ollama` /
`DEFAULT_OLLAMA_URL` / the `ollama_url` worker argument are dead plumbing that is still
threaded through the call signatures; nothing calls them. `README.md` and one NiceGUI
tooltip still describe a LLaMA stage — stale.

### Track
`worker_track` picks the backend from `state.track_backend`, falls back to TAPNext++ if
SynthEyes fails to import, and — key detail — **retries shots that produced zero tracks on
TAPNext after the whole SynthEyes pass finishes**, never inline, because SynthEyes holds
the GPU until its `finally`.

- **`app/syntheyes_engine.py`** — drives an installed SynthEyes over SyPy3. On build
  2026.2.4679 SyPy3's `ClickAndWait()` dispatch silently no-ops, so blip/peel/room-switch
  run via **Win32 `PostMessage` to the real panel buttons**, export via a **Sizzle script**,
  and SAM3 mattes are applied as a Python post-filter. SyPy3 is auto-discovered next to
  the `.exe`. No per-point loop exists here — SynthEyes blips and peels internally.
- **`app/tracker_core.py` `BatchTrackerRunner`** — the TAPNext++ path, and where nearly all
  accuracy work lives. Order per shot: seed features (edge/anisotropy rejection, staggered
  query frames) → 4 TAPNext passes (fwd/bwd/mid) → assemble → mask gating with occlusion
  continuity → quality gate + evenly-spread selection → `moving_tile_refine` (native-res
  re-track) → `pattern_refine` (3DE-style NCC + affine, sub-pixel polish, re-acquire after
  occlusion) → `track_filter` (defragment, certainty gate, backfill to floor) → export.
  VRAM safety: chunk count auto-sized from free VRAM, track IDs chained across overlapping
  seams, OOM → downscale-and-retry, per-window decode when the clip exceeds host RAM.
  Exported coords are always original plate resolution regardless of proxy scale.
- **`app/shot_profile.py`** — measures each plate (sharpness, grain, texture, motion) and
  derives the shot's parameters (`auto_tune`, on by default). Anything the user explicitly
  set is recorded in `auto_tune_overrides` and always wins.
- **`app/track_meta.py`** — per-track policy (`per_track_policy`, **on by default**, TAPNext
  only). Classifies each seed (corner/blob/edge/dense) and hands `_refine_segment` a
  read-through `_TrackCfg` view with per-track overrides. With no overrides `view()`
  returns the shot config *object itself*, so setting it False is byte-identical to the
  old single-setting path — that property is what lets one binary produce both baseline and
  treatment on identical footage for `eval_refs`. `TrackRegistry` must follow ids through
  both split sites (refine split `_b`.., defragment split `_f1`..) or every track gets its
  neighbour's settings. It exists because one shot-wide setting has no right answer on a
  plate that is sharp in one region and defocused in another.
- **Track span**: `track_filter.stitch_passes` rejoins partial tracks of the same feature
  across passes *before* the quality gate, and `pattern_refine._extend_ends` carries the
  outer ends further while the original anchor still locks. Both exist because a staggered
  seed entering late covers only the tail of the shot. Extension stops at the first failed
  frame and never enters an internal gap — that gap is an occlusion, and re-acquisition is
  what crosses it.

## Conventions

- Tracking output is 3DE ASCII with per-point frame numbers, so **gaps are legal** — an
  occluded track survives with a hole rather than being deleted.
- Frame ranges in the UI are 1-based inclusive, `0` = unset/full; exported frame numbers
  stay aligned to the original shot.
- `OPENCV_IO_ENABLE_OPENEXR=1` must be set before *any* `cv2` import (both entry points do
  this at the top of the file, above other imports — keep it there).
- Env overrides: `BTR_SAM3_WEIGHTS`, `BTR_TAPNEXT_CKPT`, `QWEN2_MODEL_DIR`, `BTR_WORK`,
  `BTR_SYNTHEYES_EXE`, `BTR_SYPY3_DIR`, `BTR_SE_PORT`, `BTR_SE_PIN`, `BTR_TDE4_EXE`,
  `BTR_MOTION_BACKSTOP`, `BTR_PORT`.
- Accuracy claims need a number from `eval_refs`/`compare_tracks` against a hand track, or
  from `bench/` against synthetic truth — "looks better in the viewport" is what this
  tooling exists to replace.
- A quality *metric* needs checking against known-good input before it is believed. Both
  defects found in 2026-08 were metrics that looked plausible and were measuring the plate
  instead of the tracker; each was caught by feeding it ground truth and seeing a number that
  should have been zero. See `bench/README.md`.

---
> Source: [antonyjaguar19-dotcom/Batch_tracker_repo](https://github.com/antonyjaguar19-dotcom/Batch_tracker_repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
