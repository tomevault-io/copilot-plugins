## sokol-net

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

Sokol.NET is a C# binding and application framework over [Sokol headers](https://github.com/floooh/sokol) — a set of single-file C libraries for graphics, audio, and application management. It targets Windows (D3D11), macOS/iOS (Metal), Linux (OpenGL), Android (GLES3), and WebAssembly (WebGL2) from a single C# codebase using .NET NativeAOT.

## One-Time Setup

```bash
# Register Sokol.NET home directory (creates ~/.sokolnet_config/sokolnet_home)
./register.sh          # macOS/Linux
register.bat           # Windows

# Ensure git submodules are initialized
git submodule update --init --recursive
```

## Build Commands

The primary build tool is `SokolApplicationBuilder` at `tools/SokolApplicationBuilder/`.

```bash
# Compile shaders only (runs sokol-shdc for all platforms)
dotnet build examples/<project>/<ProjectName>.csproj -t:CompileShaders

# Prepare project (compiles shaders, copies assets)
dotnet run --project tools/SokolApplicationBuilder -- --task prepare --architecture desktop --path examples/cube

# Run on desktop (JIT mode)
dotnet run --project examples/cube/cube.csproj

# Build/run for Android (APK)
dotnet run --project tools/SokolApplicationBuilder -- --task build --architecture android --type release --path examples/cube

# Build/run for iOS
dotnet run --project tools/SokolApplicationBuilder -- --task build --architecture ios --type release --path examples/cube

# Build WebAssembly
dotnet run --project tools/SokolApplicationBuilder -- --task build --architecture web --path examples/cube

# Serve WASM locally
dotnet serve --directory examples/cube/bin/Release/net10.0/browser-wasm/AppBundle

# Scaffold a new project
dotnet run --project tools/SokolApplicationBuilder -- --task createproject --project my_app --destination /path/to/projects
```

VS Code `.vscode/tasks.json` contains 500+ pre-configured tasks (one per example × platform).

## Rebuilding Native Libraries

Pre-built libraries live in `libs/`. Rebuild only when changing C/C++ sources in `ext/`:

```bash
./scripts/build-xcode-macos.sh            # macOS
.\scripts\build-vs2022-windows.ps1        # Windows
./scripts/build-linux-library.sh          # Linux
./scripts/build-web-library.sh            # Emscripten/WASM
./scripts/build-android-sokol-libraries.sh  # Android NDK (multi-arch)
./scripts/build-ios-sokol-library.sh all  # iOS
```

## Regenerating C# Bindings

`src/sokol/generated/*.cs` is auto-generated from C headers. Regenerate after changes to `ext/sokol/` or other C headers:

```bash
./scripts/generate-bindings.sh
# Runs bindgen/gen.py → outputs 26 .cs files in src/sokol/generated/
```

## Architecture

### Layer Stack

```
C# Applications (examples/)
        ↓
Sokol.NET C# Bindings (src/sokol/)
  ├── generated/          ← auto-generated from C headers via bindgen/gen_csharp.py
  └── hand-written/       ← helpers, extensions, utilities
        ↓
Pre-built native libraries (libs/)
  ← compiled from ext/ using platform-specific toolchains
        ↓
Platform graphics APIs (D3D11 / Metal / OpenGL / GLES / WebGL)
```

### Key Source Directories

- `src/sokol/generated/` — 26 auto-generated C# files. `SG.cs` (graphics, 178KB) and `SApp.cs` (app, 48KB) are the most important. **Do not edit manually.**
- `src/GUI/` — Custom immediate-mode GUI framework (Sokol.NET-specific, not Dear ImGui). Has its own widget, layout, binding, and theming systems.
- `src/imgui/` — Dear ImGui bindings generated from `ext/cimgui`.
- `tools/SokolApplicationBuilder/` — Cross-platform build orchestrator; handles shader compilation, asset bundling, APK/IPA creation, and WASM packaging.
- `ext/` — Git submodules for all native C/C++ dependencies. `ext/sokol.c` is the unified C compilation unit.
- `ext/CMakeLists.txt` — Single CMake file that builds all native libraries per platform.
- `bindgen/` — Python scripts that parse C headers and emit C# bindings.

### Struct-by-Value / WASM Workaround

C functions that return structs by value cannot be called directly via P/Invoke from WebAssembly. The binding generator auto-creates C wrapper functions (in `ext/sokol_csharp_internal_wrappers.h`) that return via pointer instead. See `docs/C-Internal-Wrappers-Auto-Generation.md`.

### Platform Detection

`.csproj` files use `RuntimeIdentifier` for conditional compilation. Graphics API is selected at compile time:
- `glsl430` (Linux/desktop OpenGL), `hlsl5` (Windows D3D11), `metal_macos`/`metal_ios`, `glsl300es` (Android/WASM)

### Shader Workflow

Shaders are written in Sokol's cross-compiled `.glsl` dialect and compiled by `sokol-shdc` (in `tools/sokol-tools/`) into C header files, then embedded in the C# project. Compiled outputs go to `examples/<name>/shaders/compiled/`. See `docs/SHADER_GUIDE.md`.


### Optional Dynamic Libraries

Assimp, Spine, and Ozz are loaded as separate dynamic libraries (not bundled by default). Configure via the example's `Directory.Build.props`. Assimp can have performance issues on low-tier Android; prefer cgltf or SharpGLTF for broader compatibility.

## Documentation

Full guides are in `docs/`. Key ones:
- `docs/BUILD_SYSTEM.md` — native library rebuild steps
- `docs/SHADER_GUIDE.md` — writing cross-platform shaders
- `docs/CREATE_PROJECT.md` — standalone project setup
- `docs/SOKOL_APPLICATION_BUILDER.md` — build tool reference
- `docs/VSCODE_RUN_GUIDE.md` — VS Code setup walkthrough

## Coding Principles

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- Don't assume behavior is consistent across platforms. An assumption that holds on macOS Metal may silently break on WASM/WebGL2 or Android GLES3.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
- The C#↔C interop and NativeAOT constraints already add inherent complexity — don't compound it with speculative abstractions.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- **Never manually edit `src/sokol/generated/`** — it is auto-generated from C headers. Run `generate-bindings.sh` instead.
- Changes to shared C# code or `ext/` can silently break a different target platform. Name which platforms a change has been verified on.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define acceptance criteria upfront. Loop until verified.**

Before starting, state what "done" looks like. In this project, success criteria are often visual or platform-specific rather than unit tests:
- "Fix the rendering bug" → "Cube renders without z-fighting on macOS Metal and WASM"
- "Add a GUI widget" → "Widget displays correctly and input is handled on desktop and Android"
- "Refactor shader path" → "All affected examples compile and run on all target platforms"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

When a proper automated test isn't feasible (graphics output, native crashes, platform-specific behavior), define the manual acceptance check explicitly before starting — don't leave it as "make it work".

---
> Source: [elix22/Sokol.NET](https://github.com/elix22/Sokol.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
