## displayxr-unreal

> Guidance for Claude Code (claude.ai/code) when working in this repository.

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project Overview

DisplayXR is an Unreal Engine plugin for rendering on eye-tracked 3D light field displays via the DisplayXR OpenXR runtime. The runtime handles vendor integration (interlacing, eye tracking, display processing); this plugin hooks Unreal's rendering into the runtime's OpenXR session.

Current plugin version: `DisplayXR.uplugin` → `"VersionName"`.

## Build System

No CMake — the plugin builds directly through Unreal's build system (UBT). Drop the repo into a UE project's `Plugins/DisplayXR/` folder and regenerate project files.

**Prerequisites:** UE 5.3+, Visual Studio (Windows) or Xcode (macOS), DisplayXR OpenXR runtime installed.

**Packaging for distribution** (Windows, from `Scripts/`):
```
.\PackagePlugin.bat <UE_VERSION>      # e.g. 5.6
```
Output goes to `Packages/DisplayXR_<version>/`.

## Architecture

### Three Modules

1. **DisplayXRCore** (Runtime, `Win64|Mac|Android`, `PostConfigInit`) — OpenXR integration, stereo device, camera components, Kooima C libraries, Blueprint function library.
2. **DisplayXRMaterials** (Runtime, all platforms, `Default`) — Custom material expression nodes (`StereoIndex`, `StereoSelect`, `SideBySideCoords`, `TopBottomCoords`).
3. **DisplayXREditor** (Editor, `Win64|Mac`, `PostEngineInit`) — Editor preview session, viewport widget, component proxies.

### OpenXR Integration

One unified session — `FDisplayXRSession` — loads the DisplayXR runtime **directly** on every platform (`LoadLibraryW` + `XrNegotiateLoaderRuntimeInterface` on Windows, `dlopen` on Mac/Linux). UE's `OpenXR` plugin is **not** a dependency; `DisplayXR.uplugin` declares only `XRBase`. `FDisplayXRCoreModule` implements `IHeadMountedDisplayModule` and bumps its HMD plugin priority +10 above `OpenXRHMD` / `SteamVR` so UE's HMD discovery picks us.

There is **no** `DISPLAYXR_USE_UNREAL_OPENXR` compile flag. Platform differences (DLL loading, window binding, D3D12 vs Metal graphics binding) live inside session and compositor code behind `#if PLATFORM_WINDOWS / PLATFORM_MAC / PLATFORM_LINUX`. See [`Docs/DisplayXR/Architecture.md`](./Docs/DisplayXR/Architecture.md) for the full picture and [`Docs/DisplayXR/adr/ADR-001-direct-runtime-loading.md`](./Docs/DisplayXR/adr/ADR-001-direct-runtime-loading.md) for why.

### Rendering Pipeline

`FDisplayXRDevice` extends `FHeadMountedDisplayBase` + `FXRRenderTargetManager` + `FSceneViewExtensionBase`. It:

1. Polls the OpenXR session each frame to get eye positions.
2. Feeds raw eyes through the shared `displayxr::math` Kooima C library (`camera3d_view.c`, `display3d_view.c` from the `displayxr-common` submodule) for asymmetric frustum projection.
3. Builds UE-native reverse-Z off-axis projection matrices via `DisplayXRStereoMath.h::CalculateOffAxisProjectionMatrix`.
4. UE renders directly into the OpenXR swapchain (zero-copy atlas handoff — see `Docs/DisplayXR/AtlasHandoff.md`).

### Component Hierarchy

- `UDisplayXRCamera` — Camera-centric rig. Properties: `Enable3D`, `ConvergenceDistance`, `LookaroundFactor`, `BaselineFactor`.
- `UDisplayXRDisplay` — Display-centric rig. Stereo computed relative to the display's world transform.
- `ADisplayXRRigManager` — Selects the active rig at play time.

### Public API

- `DisplayXRCamera.h` / `DisplayXRDisplay.h` — Component classes
- `DisplayXRFunctionLibrary.h` — Blueprint-exposed utility functions
- `DisplayXRTypes.h` — Shared type definitions

## Shared native code

The Kooima math (`display3d_view.{c,h}`, `camera3d_view.{c,h}`) comes from the shared [`displayxr-common`](https://github.com/DisplayXR/displayxr-common) library (`displayxr::math`), pinned as a **git submodule** at `Source/ThirdParty/displayxr-common` — run `git submodule update --init` after cloning. Don't edit the submodule contents; change the library upstream, tag, and bump the pin.

Integration mechanism (UBT has no per-file source exclusion, so the submodule must live OUTSIDE the module dirs): `Source/ThirdParty/` has no `.Build.cs` → UBT never globs it (keeping the library's `tests/selftest.c`, which has a `main()`, out of the build), while `BuildPlugin`'s package filter includes all of `/Source/...` → the submodule ships in the packaged plugin. The implementation is compiled via one-line `#include` shims — `DisplayXRCore/Private/Native/{display3d_view,camera3d_view}_impl.c` and `DisplayXREditor/Private/{display3d_view,camera3d_view}_impl.c` (Core doesn't export the symbols, so the editor module compiles its own copy; the shims must stay `.c` TUs because the implementation uses C compound literals, which C++ rejects). Both Build.cs files add `Source/ThirdParty/displayxr-common/include` to `PrivateIncludePaths` for the headers. CI lint note: the vendor-name guard greps the filesystem, but `actions/checkout` doesn't init submodules, so the submodule's copyright headers don't trip it.

## CI/CD

- `.github/workflows/lint.yml` — PR-to-main lint checks: vendor-name guard, valid JSON in `DisplayXR.uplugin`, valid YAML in workflows. Runs on `ubuntu-latest`.
- Full UE build is not in CI (UE is not on github-hosted runners). Tracked as future work in `Docs/DisplayXR/TODO.md`.

## Releasing

Preferred path: `/release` skill at `.claude/skills/release/SKILL.md`.
Tags, packages the plugin via `Scripts/PackagePlugin.bat`, creates a
GitHub Release with the packaged zip attached.

Manual fallback:
```bash
git tag -a vX.Y.Z -m "release notes ..."
git push origin vX.Y.Z
```

### Independent of the runtime's `versions.json` auto-bump matrix

The DisplayXR runtime maintains a `versions.json` at its root that
pins the **bundled stack** (runtime, shell, leia-plugin, mcp, demos)
for the dev orchestrator and the meta-installer. **This Unreal plugin
is intentionally NOT in that matrix** — it's a downstream consumer
of the runtime's OpenXR wire protocol, not part of the co-released
bundle. The Unreal sibling [`displayxr-unity`](https://github.com/DisplayXR/displayxr-unity)
shares the same boundary. See
[`displayxr-runtime/docs/specs/runtime/versions-json-autobump.md`](https://github.com/DisplayXR/displayxr-runtime/blob/main/docs/specs/runtime/versions-json-autobump.md)
for the auto-bump spec; this plugin doesn't participate.

## Documentation

- `README.md` — project overview, setup, TOC
- `Docs/DisplayXR/` — design docs (`AtlasHandoff`, `DisplayRigSetup`, `EditorPreview`, `EyeTracking`, `MacSetup`, `CompositorIntegration`)
- `Docs/DisplayXR/TODO.md` — outstanding work, parity items with `displayxr-unity`, in-flight branches

## Sibling repositories

- **[DisplayXR/displayxr-unity](https://github.com/DisplayXR/displayxr-unity)** — Unity plugin. Reference implementation; shares native C Kooima code.
- **[DisplayXR/displayxr-runtime](https://github.com/DisplayXR/displayxr-runtime)** — DisplayXR OpenXR runtime. (Old URL `dfattal/openxr-3d-display` redirects, but reference the canonical org URL in new docs.)
- **[DisplayXR/displayxr-mcp](https://github.com/DisplayXR/displayxr-mcp)** — MCP framework. Not consumed by this plugin today; future possibility if Unreal-side agent surface becomes a thing.
- **[DisplayXR/displayxr-leia-plugin](https://github.com/DisplayXR/displayxr-leia-plugin)** — Leia SR display-processor plug-in for the runtime. Vendor integration is runtime-side; this plugin doesn't talk to the SR SDK directly.
- **[DisplayXR/displayxr-installer](https://github.com/DisplayXR/displayxr-installer)** — Meta-installer bundle for end users. Bundles runtime + shell + leia + mcp + demos. This Unreal plugin is NOT bundled (see §Releasing).

---
> Source: [DisplayXR/displayxr-unreal](https://github.com/DisplayXR/displayxr-unreal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
