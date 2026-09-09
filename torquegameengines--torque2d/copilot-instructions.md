## torque2d

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Torque2D 4.0 ("Rocket Edition", Early Access) is a cross-platform 2D game engine. The C++ engine lives in `engine/source`; games are written in **TorqueScript** (`.cs` files) and structured as **modules**. The same engine binary runs the in-engine editors (Project Manager, Asset Manager, GUI Editor) and any game built on top of it.

**When writing or refactoring TorqueScript, follow the conventions in [`TORQUE_SCRIPT.md`](TORQUE_SCRIPT.md)** — the prescriptive style guide for script code (one class per file, `onAdd`/`onRemove` lifecycle, ownership/teardown chains, `class`/`superclass` inheritance).

Target platforms: Windows, macOS, Linux, iOS, Android, and Web (Emscripten).

## Building

**CMake is the single source of truth.** The engine is built from the root
`CMakeLists.txt`; you generate a project for your platform/toolchain and build it.
The executable is dropped at the **repository root** (`Torque2D.exe` /
`Torque2D_DEBUG.exe` on Windows).

- **Configure + build:** e.g. `cmake -S . -B build -G "Visual Studio 17 2022" -A x64` then `cmake --build build --config Debug` (also `Release`/`Shipping`). Single-config generators (Make/Ninja) use `-DCMAKE_BUILD_TYPE=` instead of `--config`. Convenience generator scripts live at the repo root (`generate-vs2022.bat`, `generate-vs2026.bat`, `generate-xcode.command`, `generate-make.sh`, `build-linux.sh`).
- **Per-platform recipes & status** (configure flags for macOS/iOS/Linux 32-bit/Android, runtime-verification state) are documented in `cmake/BUILD-PLATFORM-NOTES.md`.
- Engine sources are listed **explicitly** in `cmake/EngineSources.cmake` (cross-platform) and `cmake/PlatformSources.cmake` (per-platform back-ends: Windows, macOS, Linux, iOS, Android wired; Emscripten stubbed) — these are the authoritative file lists, **not** globs. (All six back-ends — Windows, macOS, Linux, iOS, Android, and Emscripten — are wired and runtime-verified.)
- Third-party libs (libogg, libvorbis, lpng, ljpeg, zlib) are built as static targets from `engine/lib/CMakeLists.txt`; GoogleTest is built via `add_subdirectory` and linked for the in-engine unit tests (desktop only).
- **Windows specifics that are load-bearing:** static non-debug runtime `/MT` for all configs (avoids `_DEBUG`, which would make tinyXML `#define DEBUG` and break Box2D), `/Zc:wchar_t-` (so `wchar_t` == the engine's `UTF16`), C++17, and `_HAS_STD_BYTE=0`.

The only remaining item under `engine/compilers/` is **not** a standalone build
system: `android-studio` is the Android app shell whose Gradle native step *invokes*
the root CMake via the NDK. The legacy hand-maintained projects — the VS solutions,
the macOS and iOS Xcode projects, the Linux Makefiles, and the Emscripten reference
recipe — have all been **retired** (the Web target is now CMake-runtime-verified);
CMake replaces them.

The built executable must run from the repo root because it loads `main.cs` and the script/asset trees (`editor/`, `library/`, `toybox/`, `tools/`) relative to the working directory.

## Running

The engine's entry point is the **`main.cs`** script next to the executable. On launch it calls `setCompanyAndProduct(...)` then `exec("./editor/main.cs")`, which starts the Project Manager UI. To boot directly into a game instead, scan and load a module (see the commented `ModuleDatabase.scanModules` / `ModuleDatabase.LoadExplicit` lines in `main.cs`).

The in-engine **console** (and the editor tabs: Asset Manager, Project Manager, GUI Editor) is opened with **Ctrl + Tilde (~)**.

## Tests

There are two suites, and they test different things.

### C++ unit tests (GoogleTest)

Vendored at `engine/source/testing/googleTest`. **Every test lives in `engine/source/testing/tests/`** (e.g. `guiTreeRowLayoutTests.cc`, `platformStringTests.cc`); each file is listed explicitly in `cmake/EngineSources.cmake`, so a new one needs a CMake edit and a re-configure to be compiled at all.

```
tests\run-unit.ps1                        all of them
tests\run-unit.ps1 GuiTreeRowLayoutTests.*  one suite (a GoogleTest filter)
```

- Under the hood that launches the engine with the alternate boot script `main.runAllUnitTests.cs`, which calls `runAllUnitTests()` and quits. You can also invoke `runAllUnitTests()` from the in-engine console.
- **`runAllUnitTests()` takes no arguments.** It hands `InitGoogleTest` an empty argv, so a subset is selected with the `GTEST_FILTER` environment variable — which is what `run-unit.ps1`'s parameter sets.
- **What a unit test can reach.** The engine boots far enough to give it `Con`, `Sim`, the string table, the resource manager and `GuiDefaultProfile`, so it can `new` and `registerObject()` a control, read and write fields, run script via `Con::evaluate`, and round-trip TAML. It has **no canvas and no GL context**, so it must never wake a control or measure text: a font registers a texture and `TextureManager::refresh` asserts — which in a debug build is a modal box, so the failure arrives as a *hang*. Note this rules out adding rows to a list box or tree, since that calls `updateSize()` → `getFont()`.
- The established move when GUI logic is worth testing is to extract the arithmetic into a `static` that takes everything it uses, then test the static — see `GuiScrollCtrl::subtractScrollBars`, `GuiControl::splitParagraphs`, `GuiTreeViewCtrl::resolveIndent`.
- Tests are compiled out of shipping builds (`TORQUE_SHIPPING` guards `unitTesting.h`).

### TorqueScript integration tests

`tests/` drives the real engine — a real canvas, the real editor, real posted mouse and keyboard input — and checks that it behaves. This is what covers the editors, which the unit tests do not reach. It is also much slower: one process per suite with a 90 second timeout each, against seconds for the whole unit run. **Prefer a unit test where the thing under test can be reached without a canvas**, and keep these for what genuinely needs one.

```
tests\run.ps1                 every pass/fail suite (exits non-zero on a change)
tests\run.ps1 colorPopup      one of them
tests\run.ps1 -Shots          the screenshot harnesses instead
```

**Read `tests/README.md` before writing one.** In particular: a relative path is expanded against the calling *script*, not the working directory, so tests use the `testRoot()` / `testExec()` helpers from `tests/lib/prelude.cs` for every path they name. Known-failing suites are recorded in `run.ps1` so a real regression stands out.

## Architecture

### Engine ↔ Script boundary (the most important pattern)
Almost every C++ class is exposed to TorqueScript. The conventions:

- A C++ class derives (transitively) from `SimObject` and uses `DECLARE_CONOBJECT(ClassName)` in its header and `IMPLEMENT_CONOBJECT(ClassName)` in its `.cc`. This registers it with the **console type system** so script can instantiate it by name (e.g. `new Sprite()`).
- Script-visible **fields** are registered in a static `initPersistFields()` override (these are also what TAML serializes).
- Script-callable **methods/functions** are defined in companion **`*_ScriptBinding.h`** files (≈140 of them) using the `ConsoleMethod` / `ConsoleFunction` / `...WithDocs` macros. These headers are `#include`d into the matching `.cc`. The doc comments in them generate the scripting reference. **When adding a script API, edit the class's `_ScriptBinding.h`, not the `.cc` directly.**

`engine/source/console/` is the TorqueScript implementation itself: lexer/parser (`CMDscan.l`, `CMDgram.y` → generated `CMDscan.cc`, `cmdgram.cc`), AST (`astNodes.cc`), compiler (`compiler.cc`, `codeBlock.cc`), and the bytecode VM (`compiledEval.cc`). Scripts compile to **`.dso`** files (gitignored); `$Scripts::ignoreDSOs` in `main.cs` controls whether compiled scripts are reused.

### Object & lifecycle core
`engine/source/sim/` is the runtime object model: `SimObject` (base), `SimSet`/`SimGroup` (containers), `SimManager` (registry, id/name lookup, event scheduling), `SimDatablock`, and script-defined objects (`ScriptObject`, `ScriptGroup`). The global `Sim` namespace owns object IDs and the event queue.

### Modules & Assets (TAML)
Games are composed of **modules**, each defined by a `module.taml` (`engine/source/module/`, `ModuleManager`/`ModuleDefinition`). A module declares a script file, create/destroy functions, dependencies, and `<DeclaredAssets>` globs. The **Asset system** (`engine/source/assets/`, `AssetManager`/`AssetDatabase`) loads `*.asset.taml` files (images, animations, fonts, sounds, particles) referenced by AssetId.

**TAML** (`engine/source/persistence/taml/`) is the object-serialization layer underpinning all of this — any `SimObject`'s persistent fields can be written/read in **XML, JSON, or binary** form. This is how editors save scenes/assets and how `.taml` files are loaded at runtime.

### 2D game framework (`engine/source/2d/`)
- `2d/scene/Scene.cc` — the world container; wraps a **Box2D** physics world (`engine/source/Box2D/`), manages SceneObjects, contacts, and the render pipeline (`SceneRenderQueue`, `SceneRenderState`).
- `2d/sceneobject/` — `SceneObject` (base renderable/physical body) and concrete types: `Sprite`, `CompositeSprite`, `ParticlePlayer`, `LightObject`, `Trigger`, skeleton/spine objects, etc.
- `2d/core/` — rendering and math support: `BatchRender` (batched OpenGL sprite rendering), `SpriteBatch`, `ParticleSystem`, `Vector2`, `ImageFrameProvider`.
- `2d/controllers/` — scene controllers (forces, e.g. point/uniform/buoyancy).
- `2d/assets/` — 2D-specific assets (ImageAsset, AnimationAsset, ParticleAsset, etc.).

### Other subsystems
- `graphics/` — `dgl` (OpenGL wrapper), texture management (`TextureManager`/`TextureHandle`), bitmap codecs (png/jpeg/bmp/pvr), fonts (`gFont`).
- `audio/` — OpenAL-based audio, Vorbis/WAV streaming, `AudioAsset`.
- `gui/` — the GUI control hierarchy (`GuiControl` and subclasses in `buttons/`, `containers/`, `editor/`). The 4.0 GUI Editor is implemented in script under `editor/GuiEditor/`.
- `platform/` + `platformWin32/`, `platformOSX/`, `platformX86UNIX/`, `platformiOS/`, `platformAndroid/`, `platformEmscripten/` — `platform/platform.h` declares the cross-platform abstraction (windowing, input, threads, file IO, networking); each `platformXXX/` provides the concrete implementation. CMake selects the right one per target.

### Script & content layout (repo root)
- `editor/` — in-engine tools (Project Manager, Asset Admin, GUI Editor, Editor Console) written in TorqueScript.
- `library/` — reusable importable modules (`AppCore`, `Audio`, `ArtPack`, …); `AppCore` provides per-project bootstrap.
- `toybox/` — 30+ example "toy" modules demonstrating engine features (good reference for script-side APIs).
- `tools/` — non-engine tooling (TexturePacker, Zwoptex, doxygen config, CMake modules, VS debugger visualizers).

## Conventions

- **TorqueScript game/UI code follows [`TORQUE_SCRIPT.md`](TORQUE_SCRIPT.md)** (repo root): one class per file named for the class, self-configuring objects via `onAdd`/`onRemove`, each object owning and deleting what it creates, and `class`/`superclass` inheritance with the `init()` pattern. Read it before touching any `.cs` game code. To verify changed scripts against these rules, use the **`checking-torquescript-conventions`** skill (`.claude/skills/`).
- Header include guards use the `_NAME_H_` convention and are wrapped in `#ifndef` checks at every include site (see `platform.h`) — follow this when adding headers.
- New engine source files must be added explicitly to `cmake/EngineSources.cmake` (cross-platform) or `cmake/PlatformSources.cmake` (platform-specific back-ends) to be compiled. These CMake lists are the single source of truth — regenerate your project from them; there are no hand-maintained `.vcxproj`/Xcode/Makefile lists to keep in sync anymore.
- All pull requests target the **`development`** branch, not `master` (master is the stable release branch).
- **A user-visible change needs a `CHANGELOG.md` entry in the same commit or PR** — new behavior, a changed or removed API, a fixed bug. Entries go under the unreleased section, written for someone building a game on the engine rather than for someone working on it. Written afterwards, they do not get written: the previous changelog lived on the wiki, was reconstructed from `git log` after each merge, and stopped in 2013.
- **`CONTRIBUTING.md`** (how to submit a change, and the C++/script standards) and **`CODE_OF_CONDUCT.md`** are at the repo root. There is no contributor agreement to sign; a contribution must simply be legally yours to give and MIT-compatible. Governance follows Torque3D's model — see `CONTRIBUTING.md` rather than looking for a steering committee charter, which was retired.
- **Documentation lives in the sibling `Torque2D.wiki` repo**, which is separate and has its own conventions — every root `.md` there is a published page. Engine changes that alter a script API or an editor workflow usually need a wiki change too.

---
> Source: [TorqueGameEngines/Torque2D](https://github.com/TorqueGameEngines/Torque2D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
