## engine

> This file provides context and instructions for AI agents working on the **Engine** project, a modular C++20 game engine.

# GEMINI.md - Engine Project Context

This file provides context and instructions for AI agents working on the **Engine** project, a modular C++20 game engine.

## Project Overview

**Engine** is a modern, high-performance C++20 game engine designed with a modular architecture. It leverages several industry-standard libraries for its core systems:

- **Rendering:** [bgfx](https://github.com/bkaradzic/bgfx) (cross-platform graphics abstraction).
- **Physics:** [Jolt Physics](https://github.com/jrouwe/JoltPhysics).
- **Audio:** [miniaudio](https://github.com/mackron/miniaudio).
- **Scripting:** [Lua](https://www.lua.org/) via [sol2](https://github.com/The_Whole_Part/sol2).
- **ECS (Entity Component System):** [EnTT](https://github.com/skypjack/entt).
- **Navigation:** [Recast Navigation](https://github.com/recastnavigation/recastnavigation).
- **UI:** [Qt6](https://www.qt.io/) for the Editor; [debug-gui](https://github.com/ocornut/imgui) (Dear ImGui) for in-engine debugging.
- **Build System:** CMake 3.21+ with [vcpkg](https://github.com/microsoft/vcpkg) for dependency management.

### Key Directories

- `engine/`: The core engine modules (e.g., `core`, `render`, `physics`, `scene`, `script`, `audio`). Each module has its own `include/`, `src/`, and `tests/`.
- `editor/`: Qt6-based editor application.
- `samples/`: Example applications demonstrating engine features.
- `tools/`: CLI tools like `engine-cli` and `image_compare` for visual regression.
- `cmake/`: Custom CMake modules and configuration files.
- `templates/`: Templates for creating new game plugins.

## Building and Running
Review the .vscode/tasks.json for how to build the different applications in the code

### Prerequisites
- C++20 compatible compiler (Visual Studio 2022/2026, GCC 13+, or Clang 16+).
- CMake 3.21+.
- vcpkg (automatically handled if `VCPKG_ROOT` is set or present in a sibling directory).
- Qt6 (for the editor).

### Running
Binaries are located in `out/build/Qt-Debug/bin/`.
- Editor: `out/build/Qt-Debug/bin/editor`
- Samples: `out/build/Qt-Debug/bin/transform_sample`, etc.

## Development Conventions

### Architecture
- **Modularity:** The engine is divided into independent modules. Prefer adding functionality to the most relevant module rather than `core`.
- **Target-based CMake:** All modules are defined as CMake targets (e.g., `engine::core`, `engine::render`). Use `target_link_libraries` to manage dependencies.
- **Headers:** Public headers should be in the `include/engine/{module}/` directory and added to the target's `FILE_SET HEADERS`.

### Coding Style
- **C++20:** Utilize modern C++ features (concepts, ranges, `std::format`, etc.).
- **Namespaces:** Most engine code lives in the `engine` namespace or module-specific sub-namespaces (e.g., `engine::render`).
- **Memory Management:** Prefer smart pointers and RAII. Use the engine's custom allocators where performance is critical (see `engine/core/allocators.hpp`).

### Testing
- **Framework:** [Catch2](https://github.com/catchorg/Catch2).
- **Location:** Tests are located in the `tests/` subdirectory of each module.
- **Registration:** Use the `engine_add_tests(MODULE_NAME)` helper in `cmake/Testing.cmake`.
- **Execution:** Run via `ctest` from the build directory or `build.bat tests`.

## Technical Notes

- **bgfx Integration:** The engine uses a custom bgfx wrapper for rendering. Most rendering logic is in `engine/render`.
- **ECS Integration:** `engine::scene` provides the `Scene` class, which wraps an `entt::registry`.
- **Scripting:** Lua scripts are typically loaded via the `ScriptSystem` in `engine/script`.
- **Game Plugins:** Games are implemented as DLLs. Use `add_game_dll()` in your game's `CMakeLists.txt` to link against the engine.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krystian-booker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
