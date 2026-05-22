## opengl-examples

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System

**Dependencies are managed via Conan.** Dependencies include glbinding, glfw, glm, fmt, tinyobjloader, imgui, nativefiledialog-extended, libuv, perfetto, and SDL2. Before running CMake, you must install Conan dependencies:

```bash
# Debug build
conan install . --settings="build_type=Debug" --build=missing

# Release build
conan install . --settings="build_type=Release" --build=missing
```

Some packages (glbinding/3.5.0, glfw/3.4, tinyobjloader/2.0.0-release, nativefiledialog-extended/1.2.1) require a forked conan-center-index. See README.md for details.

After conan install, configure and build with CMake (Ninja generator is the default):

**IMPORTANT — Windows MSVC environment:** The MSVC compiler requires `INCLUDE`, `LIB`, and `PATH` environment variables that are not set by default in bash. Before any build or test command, source the repo's `vcvars.sh`:

```bash
source ./vcvars.sh   # must be sourced before any ninja/cmake build step
```

To build:

```bash
source ./vcvars.sh
cd build/Debug   # or build/Release
cmake ../.. -G Ninja -DCMAKE_BUILD_TYPE=Debug
ninja
```

To run tests (also needs vcvars for the DLL runtime):

```bash
source ./vcvars.sh
cd build/Debug
./src/scotty3d/Scotty3D --run-tests <prefix>
```

The `vcvars.sh` script exports `INCLUDE`, `LIB`, `LIBPATH`, and adds the MSVC binary directory to `PATH`. It was generated from `vcvars64.bat` (VS 2022 Community, MSVC 14.44). If VS is updated or installed elsewhere, regenerate it by running `vcvars64.bat` in a cmd prompt, dumping `set`, and updating the paths in `vcvars.sh`.

The CMake configure step requires the conan-provided CMakePresets (`CMakeUserPresets.json` includes both Debug and Release presets).

## Project Structure

The root `CMakeLists.txt` finds all packages and delegates to `src/CMakeLists.txt`, which adds four sub-projects:

- **`src/glfw-examples/`** — Ten numbered GLFW examples (00-triangle through 09-windows). Each is a standalone executable demonstrating OpenGL/GLFW features. Lightweight, minimal dependencies.

- **`src/imgui-demo/`** — Dear ImGui demo applications. Contains a `base/` subdirectory that builds a static library (`base`) providing OpenGL wrappers (camera, shader, mesh, window, primitive drawing). Three executables depend on `base`: `test_imgui`, `obj_loader`, and `shader_toy`.

- **`src/scotty3d/`** — The main project: Scotty3D, a 3D modeling/rendering application. Uses SDL2 + imgui + glad (OpenGL loader). Architecture:
  - `lib/` — Math primitives (Vec2/3/4, Mat4, Quat, Ray, BBox, Plane, Spectrum)
  - `platform/` — OS-level glue (window, OpenGL context, console)
  - `scene/` — Scene graph, materials, cameras, lights, textures, shapes, skeletons, animation, undo, I/O
  - `geometry/` — Half-edge mesh data structure and operations, indexed mesh, splines
  - `pathtracer/` — Path tracing renderer (BVH, samplers, trace)
  - `rasterizer/` — Software rasterizer pipeline (framebuffer, sample patterns)
  - `gui/` — UI layer (managers, widgets, render viewport, rigging, simulation, animation)
  - `util/` — HDR images, RNG, thread pool, timer, viewer camera
  - `deps/` — Bundled dependencies (glad, imgui, nfd, sejp, sf_libs, perfetto)
  - `tests/` — Assignment-based tests (a0 through a4)
  - `assignments/` — Assignment writeups and resources
  - `media/` — Scene files and textures

- **`src/ecs-demo/`** — An ECS (Entity Component System) architecture demo (`nexus`). Minimal ECS framework with components (Transform, Renderable, RigidBody, etc.), systems (Render, Physics, CameraControl, PlayerControl), and glfw/glbinding rendering.

**`src/shaders/`** contains standalone GLSL fragment shaders (not compiled as part of any target — used as external resources).

## Testing

Scotty3D uses a custom test framework (see `src/scotty3d/test.h`). Tests are self-registering via global `Test` objects. Run them with:

```bash
./Scotty3D --run-tests <prefix>
# e.g.: ./Scotty3D --run-tests a2.l1
# or a single test:
#       ./Scotty3D --run-tests a2.l1.flip_edge.basic.simple
```

Tests are organized by assignment prefix: `a0.*`, `a1.*`, `a2.*`, `a3.*`, `a4.*`. Tests throw `Test::error` on failure.

## Formatting

```bash
./clang-format.sh   # runs clang-format on all tracked .cpp/.hpp/.h/.c files (8 parallel jobs)
```

The `.vscode/settings.json` expects `clangd` as the LSP with `--compile-commands-dir=${workspaceFolder}/build`. Set `C_Cpp.intelliSenseEngine` to `disabled` to avoid conflicts.

## Key Conventions

- C++17, no extensions (`CMAKE_CXX_EXTENSIONS OFF`)
- MSVC flags: `/permissive- /bigobj` with `WIN32_LEAN_AND_MEAN _CRT_SECURE_NO_WARNINGS NOMINMAX`
- ASCII-only encoding (no UTF-8/GBK/BIG-5 support)
- Custom half-edge mesh data structure is central to Scotty3D's geometry operations

## Editor Note

The Edit tool frequently fails on this repo because the code uses **tabs** for indentation, and the tool's whitespace matching is sensitive to the difference between tabs and spaces shown in the Read tool output. When Edit fails with "String to replace not found", fall back to `python` via Bash:

```bash
python << 'PYEOF'
with open('path/to/file.cpp', 'r') as f:
    c = f.read()
old = '...'  # copy the exact text from Bash: sed -n 'N,Mp' file.cpp
new = '...'
if old in c:
    c = c.replace(old, new, 1)
    with open('path/to/file.cpp', 'w') as f:
        f.write(c)
    print('OK')
PYEOF
```

Use `sed -n 'N,Mp' file.cpp | cat -A` to see the exact tab (`^I`) characters in the file before constructing the `old` string.

---
> Source: [PureWhiteVK/OpenGL-Examples](https://github.com/PureWhiteVK/OpenGL-Examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
