## hedgehogengine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System

HedgehogEngine uses **Premake5** to generate Visual Studio 2022 solution files. There is no CMake.

**Initial setup** (run once, or after submodule changes):
```bat
Scripts\SetupWindows.bat
```
This initializes git submodules recursively and generates `HedgehogEngine.sln`.

**Building:** Open `HedgehogEngine.sln` in Visual Studio 2022 and build. Configurations: `Debug` and `Release`, platform: `Windows x64`.

**Shader compilation** is automatic via a Shaders project pre-build command that calls `ThirdParty/glslc/CompileShaders.bat`, compiling `.vert`/`.frag`/`.comp` GLSL sources to SPIR-V (`.spv`).

**Build output:** `Binaries/Windows-x64/[Debug|Release]/[ProjectName]/`

**Running tests:**
```
Binaries/Windows-x64/Debug/HedgehogMathTest/HedgehogMathTest.exe
```

**Regenerating solution** after modifying any `Build.lua` files:
```bat
cd Scripts && premake5 --file=..\Build.lua vs2022
```

## Architecture

The engine is a collection of C++20 static libraries with a single `Client` console application as the entry point. Dependencies flow strictly upward:

```
Client
  └── HedgehogContext + HedgehogRenderer + Logger
        ├── HedgehogContext → HedgehogCore, HedgehogSettings, HedgehogWrappers,
        │                     ContentLoader, DialogueWindows, Scene, yaml-cpp, Vulkan, ImGui
        ├── HedgehogRenderer → HedgehogContext, HedgehogWrappers, Shaders, Vulkan, ImGui
        ├── HedgehogWrappers → HedgehogCore, HedgehogMath, GLFW, Vulkan
        └── HedgehogCore → HedgehogMath
```

### Key Modules

| Module | Type | Role |
|--------|------|------|
| `HedgehogMath` | static lib | Vectors, matrices, AABB/OBB/Plane/Frustum primitives |
| `HedgehogCore` | static lib | Camera, common utilities |
| `HedgehogWrappers` | static lib | GLFW window, raw Vulkan object wrappers |
| `HedgehogSettings` | static lib | YAML-based engine configuration |
| `HedgehogContext` | static lib | Vulkan/Engine/Frame/Thread context; resource containers (DrawList, Light, Material, Mesh, Texture) |
| `HedgehogRenderer` | static lib | Multi-pass Vulkan renderer (see render passes below) |
| `ECS` | static lib | Entity Component System (EntityManager, ComponentManager, SystemManager, Coordinator) |
| `Scene` | static lib | Scene graph, ECS-based components/systems, YAML serialization |
| `ContentLoader` | static lib | glTF/glb, OBJ, and texture loading (stb_image) |
| `DialogueWindows` | static lib | ImGui-based dialogs for materials, meshes, scenes, textures |
| `Logger` | static lib | Colorized console logging, no dependencies |
| `Shaders` | static lib | GLSL sources; compiled to SPIR-V by pre-build step |
| `Client` | executable | Application entry point (`HedgehogClient` class) |

### Render Passes (HedgehogRenderer)

Forward-rendering pipeline executed in order:
1. **InitPass** — initialization
2. **DepthPrepass** — early-Z depth pass
3. **ShadowmapPass** — shadow map generation
4. **ForwardPass** — main lit geometry pass
5. **ShadowsAddingPass** — shadow composition
6. **GuiPass** — ImGui overlay
7. **PresentPass** — swapchain presentation

### Project Configuration Files

Each module has its own `Build-[ModuleName].lua` file included from the root `Build.lua`. Global third-party paths and library names are centralized in `Dependencies.lua`. Modifying either requires regenerating the solution.

### Testing

Only `HedgehogMath` has unit tests (`HedgehogMathTest`), using the **doctest** framework. Test files: `test_vector.cpp`, `test_matrix.cpp`, `test_common.cpp`. Floating-point comparisons use `NearlyEqual` helpers with configurable epsilon.

### Third-Party Dependencies (git submodules)

glfw, ImGui, yaml-cpp, tinygltf, doctest, Lua — all under `ThirdParty/`. Vulkan SDK headers/libs are also under `ThirdParty/vulkan/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vVoitovych) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
