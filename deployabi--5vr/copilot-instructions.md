## 5vr

> 5VR is a single-player-only VR injection mod for RAGE-engine games

# Repository Guidelines

## Mission & Non-negotiables
5VR is a single-player-only VR injection mod for RAGE-engine games
(GTA V Legacy first). Hard rules, enforced in code and review:
- **Story Mode only.** Online sessions (GTA Online) and BattlEye presence
  hard-disable the mod via `OVRInject/Game/OnlineGuard` (ADR-0003). There is
  no config bypass; never add one; never weaken it "for testing".
- **No DRM/anti-tamper circumvention** and no committed game assets or
  third-party mod material. Prior-art/reference material from other mods is
  never committed (see `docs/LEGAL.md`).
- **Version-pin everything:** AOB patterns, offsets, and shader hashes live in
  `manifests/*.ini`, never hardcoded in source (ADR-0004).
- Label `UNVERIFIED` anything not confirmed on real hardware in a real headset.
- Design decisions live in `docs/01-architecture.md` + `docs/ADR/`; update them
  when behavior/conventions change.

## Project Structure & Module Organization
- `GTAVOVR.sln` hosts the Visual Studio solution (projects: GTAVOVR, OVRInject,
  OVRInjectShim; slice + tests projects are being added).
- `GTAVOVR/` is the dev launcher (env setup + CreateRemoteThread injection;
  target process via argv[1] or `GTAVR_TARGET_PROCESS`). `--check` runs
  preflight only (disabled marker, build detection vs `manifests/`, VR
  runtime, BattlEye posture); exit codes 0 ok-to-try, 2 unsupported build,
  3 runtime missing, 4 injection failure, 5 mod disabled by `gtavr.disabled`.
- `OVRInject/` holds the core VR injection logic, D3D hooks, OpenXR/OpenVR
  backends, overlay UI, and shaders. Subdirs: `D3DHook/` (Present/ResizeBuffers
  hooks, dummy-device late-injection, HudRedirect), `VR/` (backend
  abstraction), `OpenXR/`, `Game/` (title plugin: camera/FOV/state, OnlineGuard,
  BuildManifest — all pattern resolution on a bounded background worker; the
  render thread is O(1)), `Stereo/` (StereoEngine + EyeDelivery AER state
  machine, ComfortRuntime), `Perf/` (PerfStats frametime ring + CSV export),
  `Overlay/`, `Vive/` (HMDRenderer only).
- `OVRInjectShim/` is the shipped loader: a proxy `dxgi.dll` that loads
  `OVRInject.dll` (ADR-0005).
- `tools/` holds `install.bat` / `uninstall.bat` (idempotent game-dir
  installer/reverser via `runtime_package.ps1`; install/verify refuse game
  builds the pinned ScriptHookV cannot load — `OfficialGameBuild` in
  `tools/launcher/ScriptHookPackage.cs` — or builds with no manifest
  section; smoke-test against throwaway dirs only),
  `bootstrap_thirdparty.ps1` (hash-verifies/fetches the vendored libs a
  clean clone needs),
  `GTAVR-Play.bat` (one-click game launcher), `GTAVR-Panel.bat` +
  `gtavr_panel.py` (control panel: start game, backend switch, inject, live
  log tail), `collect_logs.bat/.ps1` (support bundle to Desktop),
  `scan_patterns.py` + `verify_camera_chain.py` (new-build pattern
  resolution against the running game).
- `samples/D3D11Cube/` is the Phase 2 vertical-slice target + golden-image
  harness (`check_golden.py`).
- `tests/` holds the unit-test runner (38 suites incl. TestEyeDelivery;
  402 tests / 75111 checks green on 2026-08-11) and the
  injection-lifecycle harness (`injection_lifecycle.py`).
- `manifests/` holds per-title, per-build signature/offset/hash INIs.
- `docs/` holds feasibility, architecture, ADRs, legal, known-issues,
  test-matrix, perf-report, hud-postfx, and `docs/user/` guides.
- `ThirdParty/` is vendored dependencies (OpenVR, OpenXR, ImGui, minhook,
  DirectXMath) — avoid edits unless required.
- Root `.ini` files (`gtavr_camera.ini`, `gtavr_settings.ini`) are templates
  for runtime config; built artifacts go under `x64/` and `build_*/` (ignored).

## Build, Test, and Development Commands
- Visual Studio: open `GTAVOVR.sln`, build `Release|x64` (toolset v145).
- CLI build (Git Bash — use dash switches; `/p:` gets path-mangled):
  `env -u GTAV_INSTALL_DIR "/c/Program Files/Microsoft Visual Studio/18/Community/MSBuild/Current/Bin/MSBuild.exe" GTAVOVR.sln -p:Configuration=Release -p:Platform=x64 -m -v:m`
  (`env -u GTAV_INSTALL_DIR` is required: the post-build copy into the game
  directory fails the build if that env var points at a non-writable path.)
- Unit tests: build `tests/GTAVRTests.vcxproj` and run the exe
  (`tests/run_tests.bat`).
- Slice golden test: `GTAVR_SLICE_GOLDEN=40 D3D11Cube.exe` twice, then
  `python samples/check_golden.py` (determinism + stereo disparity).
- Prereqs: OpenVR headers in `ThirdParty/openvr/headers`, OpenXR SDK per
  `ThirdParty/openxr/README.md`, ImGui per `ThirdParty/imgui/SETUP.md`
  (`HAS_IMGUI`).
- Clean-clone bootstrap: the build links vendored binaries
  (`ThirdParty/openvr/lib/x64/openvr_api.lib`,
  `ThirdParty/openxr/lib/x64/openxr_loader.lib`, plus the runtime
  `openvr_api.dll` / `openxr_loader.dll` payloads), which are committed so a
  clean clone has them. The non-redistributable ScriptHookV SDK import
  library (`GTAVRBridge/ScriptHookV.lib`) is deliberately NOT committed —
  building the GTAVRBridge project needs a manual fetch. If anything is
  missing or fails verification, run
  `powershell -ExecutionPolicy Bypass -File tools/bootstrap_thirdparty.ps1`
  — it hash-verifies (SHA-256, pinned) what is present, re-fetches OpenVR
  v2.12.14 and the OpenXR loader 1.1.54 from their pinned upstream URLs, and
  prints manual fetch instructions for the ScriptHookV SDK import library.
  `-VerifyOnly` checks without downloading.

## Coding Style & Naming Conventions
- C++ with `.hpp`/`.cpp` pairs; keep includes and precompiled headers
  consistent with nearby files.
- Follow local formatting; common pattern is 4-space indents and braces on
  the same line.
- Naming: `PascalCase` for classes/functions, `lowerCamelCase` for locals,
  trailing `_` for members (e.g., `device_`), ALL_CAPS for macros.
- The injected DLL must never terminate the host game: no `exit()` paths;
  log at ERR and degrade to pass-through instead.

## Testing Guidelines
- Automated: matrix-math unit tests (`tests/`), slice golden/disparity
  harness (`samples/`), injection-lifecycle and soak harnesses (see
  `docs/test-matrix.md`).
- Manual validation is in-game, per `docs/test-matrix.md` — a scenario may
  not be marked PASS without a capture/trace attached; frametime evidence is
  p99/p99.9, never averages.
- Vendor tests in `ThirdParty/readerwriterqueue/tests` are not part of the
  project test flow.

## Commit & Pull Request Guidelines
- Commit history uses short, sentence-case summaries; keep messages concise
  without prefixes. Phase work commits per phase with evidence (test output,
  capture references) in the message body.
- Never stage gitignored reference/prior-art material (see Mission above).
- PRs should include a summary, affected modules (e.g., `OVRInject/OpenXR`),
  runtime tested (OpenXR/OpenVR), and any config/env var changes.
- Include a screenshot or short clip for overlay/UI changes when possible.

## Configuration & Runtime Tips
- Select runtime with `GTAVR_BACKEND=openxr|openvr` (OpenXR primary,
  ADR-0006); `XR_RUNTIME_JSON` can point to a specific OpenXR runtime.
  Experimental, default-OFF: `GTAVR_VR_GAMEPAD=1` (VR-controller virtual
  gamepad via an XInput detour) and `GTAVR_GAMEPLAY_CAM_SYNC=1`
  (snap/smooth-turn gameplay-camera yaw resync) — both UNVERIFIED on
  hardware, see `docs/known-issues.md`.
  Diagnostics, default-OFF, **INI-keys in `gtavr_settings.ini`** (env vars do
  NOT arm them since 2026-08-07 — a stale process env inherited from a
  long-deleted `setx` re-triggered the spin/warp/dump diagnostics in real
  user sessions three separate times, so env arming was removed entirely;
  use a scratch `GTAVR_SETTINGS_DIR`
  copy for harnesses): `depthViz=1` (Phase-0.5 probe:
  captured scene depth linearized as grayscale into the right eye) with
  `depthVizInvert=1` to flip the depth convention;
  `headSpinDegS=<deg/s>` (synthetic constant-yaw rotation + slow positional
  sway injected into the latched head pose — drives the
  camera/capture/submit pipeline without a human, for submitted-frame dump
  analysis); `eyeDumpDir=<dir>` with `eyeDumpEvery=<n>` (dumps a 1024px
  center crop of the submitted post-warp swapchain texture as BMP every
  n-th submit per eye); `depthProbe=1` (same-size non-MSAA DSV bind
  instrumentation); `sceneCapture=1` (opt-in final-image capture);
  `depthWarpExperimental=1` (PoseWarp v2 depth warp arm).
  `GTAVR_LOG_MAX_BYTES` overrides the 16 MiB log-rotation cap (clamped
  to [1 MiB, 1 GiB]; current file plus `.1`/`.2` are kept) in both
  `OVRInject` and `OVRInjectShim`.
- Config files `gtavr_settings.ini` and `gtavr_camera.ini` live under
  `GTAVR_SETTINGS_DIR` (default: `%LOCALAPPDATA%\GTAVR` — user-writable,
  so overlay/panel changes persist even though the game dir is
  admin-only; legacy behavior searched beside `GTA5.exe`). Notable keys:
  `eyeTargetMaxDimension` (hard cap on the resulting per-eye target size,
  default 4096 px, 0 = off — clamps the 2x-panel "quality" waste that
  collapses frametimes; the overlay shows "eyeTargetMaxDimension bound"
  when it binds), `rotationWarp` (in-app render→fresh rotation warp at
  submit, default 1 — on OpenXR it warps into the swapchain image before
  layer submit; since 2026-08-09 it also arms the OpenVR pre-submit
  late-warp, which warps into the per-eye submission texture on both the
  legacy copy path and the direct DXGI 1.1+ path — watch the 1 Hz
  `[warpdiag]` line for validation) and `translationWarp` (PoseWarp v2:
  depth-based rotation+translation reprojection per submitted eye, default
  1 — falls back to rotation-only for eyes without matching scene depth;
  OpenXR only).
- Override log location with `GTAVR_LOG_PATH` or `GTAVR_LOG_DIR`.

---
> Source: [DeployAbi/5VR](https://github.com/DeployAbi/5VR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
