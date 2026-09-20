## pz-optimization

> Class overrides for Project Zomboid Build 42 (Java, LWJGL/OpenGL) that improve chunk

# PZ_Optimization

Class overrides for Project Zomboid Build 42 (Java, LWJGL/OpenGL) that improve chunk
streaming and driving frame time, plus a hands-off benchmark harness that measures them.
Public repo (xD3I/PZ_Optimization). Maintainer: the repository owner (they/them).

## Objective (stated 2026-09-18)

"Consistent frame time if the CPU and GPU utilization allows it; the CPU and GPU should
always be used to the max if the framerate is not smoothly pegged at 240 fps."
Every benchmark report must show frame-tail metrics (p99 / p99.9 / spikes / jitter) AND
utilization (CPU/GPU load from sysmon) over the route window. "fps < 240 and hardware not
saturated" is itself a finding. Chunk-latency wins are done; do not spend more on the streamer.

## Hard rules

- **`decompiled/` stays local** (gitignored, 24 MB CFR output of the whole jar). Never `git add`
  it. `src/overrides/` (the 25 shadowed classes with our `// pzopt:` edits) IS committed since
  2026-09-19: the maintainer confirmed the sources may ship. Every edit is still described in prose in
  `docs/override-edits.md`, and the `// pzopt:` markers stay on every changed line.
- **Shared machine.** Several Claude sessions and the maintainer use the one game install and `~/Zomboid`.
  Before a launch or a reinstall check both `pgrep -f '[P]rojectZomboid64'` and
  `pgrep -f '[h]arness/run.sh'` (excluding your own). Message busy peers (ListAgents /
  SendMessage) before reinstalling or starting a batch. See `.claude/skills/bench-run`.
- **Run etiquette.** The maintainer is usually at the machine. Say a run is about to start before
  launching, one run at a time, never long batches. Never edit `harness/run.sh` while a run is in
  progress (bash reads it incrementally; a mid-edit launch died and its EXIT trap corrupted
  `latestSave.ini`). Launch from a copy (`harness/.run-snapshot.sh`) if someone else edits it.
- **Never kill with a self-matching pattern.** `pkill -f '<pattern>'` where the pattern appears in
  your own command line kills the tool shell (exit 144). Use bracket patterns like
  `[P]rojectZomboid64` or a saved PID.
- **All recordings and videos are AV1 HDR.** Every `--record` capture (gpu-screen-recorder
  `-k av1_hdr`: AV1 10-bit PQ / BT.2020) and every video published under `docs/media/` (the
  stitch scripts, `harness/encode-av1-hdr.sh`, the `-1080` README copies) is encoded AV1 10-bit
  with PQ / BT.2020 tags; never tone-map to SDR H.264. SDR sources are mapped to PQ; posters and
  GIFs are the only tone-mapped derivatives. See `harness/CLAUDE.md`.
- **Real saves are never loaded or written** by a harness run. Runs use the copied bench save
  `Saves/Sandbox/pzopt-bench` and must quit on their own. Launching the game via
  `harness/run.sh` is authorized without asking.
- Commit and push only when asked. Commits end with
  `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.

## Environment facts

| Item | Value |
|---|---|
| OS | CachyOS (Arch), fish shell, `paru` for AUR, 16 cores, 30 GB, RTX 4090 |
| JDK | system `jdk-openjdk` 26; game bundles its own JRE (build.sh compiles with `--release` for it) |
| Game dir | `/games/steamapps/common/ProjectZomboid/projectzomboid/` (native Linux depot since 2026-09-18) |
| User dir | `~/Zomboid` (console.txt, Saves, Lua/, mods/, options.ini) |
| Layout detection | `scripts/pz-env.sh` (PZ_DIR / ZOMBOID env override) |
| Desktop | 5120x2160, 240 Hz; the game renders windowed at desktop resolution |
| Decompiler | CFR at `~/.local/share/java/cfr.jar` for reading; Vineflower for overrides |
| Code index | `.codegraph/` exists; use `codegraph_explore` before grep/Read |
| Java LSP | `jdtls` not installed (check `command -v jdtls` before relying on the LSP tool) |

## Reading game code

Read game classes from `decompiled/` (CFR output of all 3,407 game classes, package tree such as
`decompiled/zombie/iso/IsoChunk.java`). Never re-decompile or unzip the jar for those packages.
Only two methods lack bodies (`CompressIdenticalItems.areItemsIdentical`,
`ChooseGameInfo.readModInfoAux`); use `javap -c -p` for them. If the jar mtime changes (game
update) re-run `scripts/decompile.sh` and `scripts/regen-overrides.sh`.

## Layout

| Path | Purpose | Details |
|---|---|---|
| `src/` | overrides, shims and the `pzopt` helper package | `src/CLAUDE.md` |
| `scripts/` | build / install / decompile / test | `scripts/CLAUDE.md` |
| `harness/` | run.sh, analysis scripts, baselines, run outputs | `harness/CLAUDE.md` |
| `config/` | MangoHud profiles | `config/CLAUDE.md` |
| `docs/` | plans, findings, override edit log, dashboard | `docs/CLAUDE.md` |
| `tools/` | standalone Java probes (JFR dump, GLFW swap probe, static audit) | `tools/CLAUDE.md` |
| `tests/` | JVM-only unit tests for pzopt classes (`scripts/test.sh`) | |
| `decompiled/` | CFR output, local only | |
| `openspec/` | OpenSpec change proposals (opsx skills) | |

## Skills (in `.claude/skills/`)

| Skill | Use when |
|---|---|
| `bench-run` | launching any measurement run (bench / drive / parity / verify) |
| `showcase-drive` | recording the stock-vs-optimized drive videos and the quad stitch |
| `build-install` | compiling the overrides and installing them into the game dir |
| `release-windows` | building the Windows zip and publishing it as a GitHub release asset |
| `analyze-run` | reading a finished run: analyze, compare, waits, loadtime, dashboard |
| `override-game-class` | adding or changing an overridden game class |
| `game-update` | the jar changed: re-decompile, regen overrides, rebuild, re-baseline |
| `proton-run` | preparing a Windows/Proton comparison run |
| `mangohud-overlay` | HUD/CSV problems, launcher and display-server hooks |

## Current state (2026-09-19)

- Adopted Config defaults (max-zoom route mean 6.2 → 4.4 ms, p99 19.3 → 8.3):
  treesInChunkTexture, windowsInChunkTexture, bakeBudget=8 (never-baked
  levels only), lightingBudget=8 (queued, never drops JNI dirty bits), hotsaveIntervalSec=30,
  on top of wake + recalc pool. persistentVbo and translucentTilesInChunkTexture are ON by
  default since 2026-09-20 afternoon (maintainer's decision after confirming the fix): the
  chunk-sized black squares seen with them were `lightInfoChunkGate` leaving never-cached squares
  out of the bake (fixed; persistentVbo only changed the timing). The 2026-09-19 reports (black
  building lot, Translucent tiles baking black) were not reproduced on the bench route. Rig:
  `run.sh --shot-at N` + `harness/blacktiles.py`; 0 black tiles after the fix, uncapped 512 fps.
  Black one-tile rectangles beside walls were cutawayFast replaying the stock int-shifted
  occluder mask (fixed with an exact mask); JUMBO trees missing near buildings were the
  FBORenderTrees batch in chunk-texture mode (trees now bake via the plain sprite path).
- Verify runs on a copy of a real save: `--source-save Apocalypse/<name>` (template kept under
  Saves/<mode>/pzopt-template-<name>); auto-start presses click-to-start and marks started
  in every mode, so the run exits to desktop after `--quit-after`.
- Remaining tail with the PZDashboard mod is its 2 s collectors; measure with `--no-dashboard`.
- Uncapped: NVIDIA GL is GPU-bound (98 %) at 570 fps; Zink blocks ~1.8 ms/frame in swap.
  The in-game limiter is stock again; "Uncapped" is a real Display-options entry and a
  second "Menu framerate" combo caps the menus separately (`pzopt.FrameCap`, Lua under
  `src/lua/`, setting in `~/Zomboid/pzopt/framecap.ini`). Uncapped runs: `--prop uncappedFps=true`.
  `uiRenderOffscreen=true` in options.ini removes the per-frame Lua UI draw. Both combos also
  offer 500/430/400/330/300 fps; the in-game choice is snapshotted before `Core.loadOptions`
  rewrites options.ini and a cap above 244 lives in framecap.ini (`gameFps=`); forced
  `uncappedFps=` runs restore the player's choice on the next boot (`restore=`).
- Options > Optimizations tab (2026-09-20): every Config key as a tick box / combo
  (`src/lua/client/pzopt/pzopt_optimizations_options.lua`, Java side `pzopt.UserOptions` +
  `PerformanceSettings` forwards), saved to `~/Zomboid/pzopt/options.ini`, applied on the next
  launch; `-Dpzopt.*` and the game dir's `pzopt.properties` (harness `--prop`) still win, so runs
  never depend on menu choices.
  Master switch `enabled` (2026-09-20 evening): `enabled=false` folds into `Overrides.enabled()`,
  i.e. the build-mismatch stock path everywhere; tab buttons "Disable all (stock game)" /
  "Enable all (recommended defaults)". `--prop enabled=false` is a stock run without a reinstall.
- Game thread (2026-09-20, `docs/results.md`): new heavy bench route `--flag route=S:450 --flag turn=90
  --route-seconds 25` (south through Rosewood, facing spinning). Adopted: weatherMaskIdleSkip,
  cutawayRadius=6, gridStackInterval=8, lightingRebakeMs=250, rebakeBudget=4/rebakeMaxFrames=3,
  lightSwitchCheckFrames=15, single-lookup Kahlua rawget, occluder masks on IsoChunk. 199 → 229 fps
  there, 230 → 238.5 on the 100 s route. No gain from bakeBudget=3, uiRenderOffscreen or
  lightingRebakeMs=1000. Game thread now 97 % busy with broad work (bakes 20 %, world update 23 %,
  Lua UI 10 %): 240 locked on that route needs a structural change, not more trims.
- In-game performance overlay (2026-09-20, `pzopt.Overlay`, F9 or `--prop overlay=true`): presented
  frame time, p99/p99.9/max/1%-low/jitter/spikes over 5 s, GPU busy (GL timer query), game/render
  thread load, verdict line, frame graph; every harness run also writes `pzopt-overlay.out`
  (MangoHud columns + epoch_ms) and `analyze.py` prints it as `overlay:`. Same numbers on
  Windows/Linux without MangoHud or RivaTuner. The stock "Display FPS" graph (K) is debug-only bars.
- Uncapped 400 fps pass (2026-09-20 evening, `docs/plan-400fps.md`, runs `u400-*`, in-game overlay log
  only, `--no-mangohud`): spinning route 273 → ~500 fps mean, p99 13.2 → 7.7 ms. Stock option
  `uiRenderOffscreen=true` is +40 % uncapped (runs pass `--option uiRenderOffscreen=true`). Adopted
  keys: cutawayInvalidateChanged, cutawayVisitPrefilter, lightInfoOncePerFrame, lightInfoChunkGate,
  occlusionSkipLightingOnly, soundZoneCache, chunkHandoffDivisor=8. Dead ends: weatherFxScalePct,
  lightingRebakeMs=1000, bakeBudget=4, hotsaveStaged (off: cross-file consistency). GPU is the wall
  from ~450 fps (`pzopt.GpuSections`, `--prop gpuSections=true`: chunk composite ~0.6 ms, bakes
  ~0.3-0.5 ms a frame). `harness/gametree.py` prints the game-thread call tree from a JFR run.
  Never build or decompile while a run is going; check `pzopt.sh status` says installed before a launch.
- JVM matrix on the desktop (2026-09-20 13:35, `docs/results.md`): GraalVM 25.0.3 is 7-9 % behind
  Zulu/C2 uncapped on the spinning route (copy kept at `jre64_graal`); Zulu + the tuned G1 JSON
  (`config/launcher/ProjectZomboid64.g1.json`, now installed) has the tightest tail (508 fps, p99 7.3 ms,
  game thread 81 %). The ~500 fps numbers need `persistentVbo=true translucentTilesInChunkTexture=true`
  (tab file or `--prop`); with both off the same route is 184 fps, so check the console `settings:` line.
- Thunderstorm pass (2026-09-20 evening, `docs/findings-scene-presets-2026-09-20.md` §3-5): the storm
  preset was 83 fps uncapped with nothing saturated. GameProfiler A/B (`--game-profiler`, `sections.py
  --thread game|render`, the file names are `MainThread` = game thread, `main` = render thread) and JFR
  showed puddles (4.5 ms/frame, stock re-packs every wet square) and the rain quads (73 % of the render
  thread, `VBORenderer` flushing every 28 quads). Adopted: `puddleCache` (`pzopt.PuddleCache`, packed
  puddle vertices per chunk level, lights/jiggle/depth patched per frame; `IsoPuddles` override, slot
  on `IsoChunk`) and `vboBatchKb=1024` + `vboFastQuads` (`VBORenderer` override). Storm route 83 → 131
  fps, p99 43 → 25 ms. Left: the rain particle path (~100k quads a frame at 5120x2160, walked twice on
  the game thread), splashes, `GameWindow.logic`. The 15:49 build with the VBORenderer change is on a
  peer session's suspect list for interior-object flicker; recordings `storm-rec-cur` vs
  `storm-rec-vbostock` are the A/B.
- Rain tiles + re-bake spread (2026-09-20 night, findings §5): `rainTiles` (`pzopt.RainTiles`,
  `ParticleRectangle` + `WeatherParticleDrawer` overrides: particle cell packed once, drawn once per
  screen cell) → desktop spinning storm 111 → 188 fps, laptop storm drive 84 → 106. The rain
  "vanishing" every 6 s in storms was five 50-90 ms stalls per lightning strike (all flash-dirtied
  chunk textures re-baked in one frame); lighting-only re-bakes now spread with
  `lightingRebakeBudget=8` / `lightingRebakeMaxFrames=30` (storm drive p99.9 57 → 12.5 ms, faint
  chunk checkerboard for ~90 ms while a flash ramps). Laptop runs go through
  `/tmp/pzopt-laptop-run.sh` on diego-flip (`~/PZ_Optimization-rain` worktree). The interior-object
  flicker itself was the peer's find, fixed in 100f441 (held re-bakes drew empty per-frame lists).
- Flicker fix (2026-09-20 evening, `docs/results.md`): objects inside buildings, doors, windows and
  corpses blinked out for 1-3 frames because stock `FBORenderLevels.invalidate()` empties the per-frame
  square lists and every pzopt held re-bake (`lightingRebakeMs`, `rebakeBudget`) drew the previous
  texture with them empty. FBORenderCell now keeps the lists across invalidations (IsoChunk clears them
  on pool reuse) and never holds cutaway re-bakes. Repro/metric: `run.sh --flag hold=10` + `harness/flicker.py`
  (runs `flick-*`); stock 3.8 vs broken 26 transient px/frame at `--scale 2560`.
- Open plans: `docs/plan-game-load.md`, `docs/plan-vulkan-renderer.md`, `docs/plan-resource-use.md`,
  `docs/plan-400fps.md` (the locked-400 structural items), `docs/plan-500fps.md` (what is
  still untouched: character update/animation, sprite recording, vispoly, Lua UI, render thread).
- Native Wayland works via `--env JAVA_TOOL_OPTIONS=-Dzomboid.wayland=1`; A/B on 2026-09-19 is a
  wash at the 240 cap (XWayland stays default; the NVIDIA GL worker thread only exists under
  GLX). `docs/plan-wayland.md`.
- Proton run prepared but blocked on the maintainer forcing a compat tool in Steam.
- Boot/load (2026-09-19 evening, `docs/plan-instant-load.md`): launch → menu 7.35 → 5.00 s,
  Continue → world 6.53 → 4.03 s, via boot threads (FMOD, anim sets), a boot-time file-pool
  pump, Lua precompile, animation clip + pack index caches (`~/Zomboid/pzopt/`), linear script
  parser, `Item.DoParam` switch, loader memos. First boot after a cache wipe is slower.
- Issue #1 (laptop, 2026-09-19 night): 16.5 s of its load was `Model.CreateShader` blocking on
  the render thread once per model (73 animal models × one ~220 ms loading-screen step). The
  `Model` override serves repeat shaders from `pzopt.ModelShaders` (`shaderCache`);
  `loadtime.py` row C2a shows the window. Why that laptop's loading-screen step is 220 ms is open.

---
> Source: [xD3I/PZ_Optimization](https://github.com/xD3I/PZ_Optimization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
