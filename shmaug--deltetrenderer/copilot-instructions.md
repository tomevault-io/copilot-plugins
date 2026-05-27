## deltetrenderer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DelTetRenderer (vulkanrm) is a Vulkan-based real-time renderer for Delaunay tetrahedral meshes. It renders `ckpt.ply` files produced by the [Radiance Meshes](https://github.com/half-potato/radiance_meshes) training pipeline. Written in C++23 with Slang shaders.

## Build Commands

```bash
# Ubuntu dependencies
sudo apt install build-essentials libxrandr-dev libx11-dev libvulkan-dev libxinerama-dev libxcursor-dev libxi-dev libxcb-keysyms1-dev

# Build
cmake -S . -B build
cmake --build build
```

Produces two executables in `build/bin/`:
- **DelaunayTetRenderer** — Interactive viewer/editor
- **benchmark** — Headless benchmarking with COLMAP camera paths

```bash
# Run interactive viewer
./build/bin/DelaunayTetRenderer

# Run benchmark (example)
./build/bin/benchmark --scene ckpt.ply --colmap /path/to/sparse/0/ --auto -d 1 -r 1080p

# Run full benchmark suite across scenes
./scripts/download_n_benchmark.sh <version_suffix>
```

## Architecture

### Namespace & Core Classes

Everything lives in the `vkDelTet` namespace.

- **`DelaunayTetRenderer`** (`src/DelaunayTetRenderer.hpp`) — Top-level orchestrator. Holds a tuple of renderer backends and dispatches to the active one via compile-time tuple indexing (`CallRendererFn`), avoiding virtual functions.
- **`RenderContext`** (`src/RenderContext.hpp`) — Shared GPU state: sort buffers, culling buffers, evaluated colors, render target, camera. Owns the compute pipelines for culling, sorting, SH evaluation, and alpha inversion.
- **`TetrahedronScene`** (`src/Scene/TetrahedronScene.hpp/.cpp`) — CPU-side source of truth (vertices, tet indices, densities, gradients, SH coefficients) with lazy GPU buffer allocation. Loads PLY files via tinyply. Supports sparse GPU updates.

### Renderer Backends (src/Renderers/)

Five interchangeable renderers selectable at runtime via ImGui dropdown:
- **MeshShaderRenderer** — GPU mesh shaders (modern hardware)
- **RasterRenderer** — Traditional vertex/fragment rasterization
- **InstancedRenderer** — Instanced tetrahedron drawing
- **PointCloudRenderer** — Point-based visualization
- **BillboardRenderer** — Sprite/billboard approach

Each renderer has a `.hpp` (C++) and `.3d.slang` (shader) file. All share the same `RenderContext` and follow a common interface: `Name()`, `Description()`, `DrawGui()`, `Render()`.

### Rendering Pipeline

1. **Frustum Culling** (`Culling.cs.slang`) — mark, prefix-sum scan, scatter
2. **Depth Sorting** (`TetSort.cs.slang`) — Radix sort by circumsphere power from camera (uses RoseLib's GPU radix sort)
3. **SH Evaluation** (`EvaluateSH.cs.slang`) — Evaluate spherical harmonics at camera position
4. **Rasterization** — Renderer-specific pass with custom blend state for order-independent transparency
5. **Alpha Inversion** (`InvertAlpha.cs.slang`) — Post-process: T = 1 - product of individual transmittances

### Interactive Editing (src/Gizmos/)

- **VertexSelection** — GPU-based point picking with depth cycling (right-click to select, scroll to adjust depth)
- **PBDMove** — Position-Based Dynamics deformation with distance and volume constraints
- **LagragianMove** — Direct Lagrangian vertex translation
- Keyboard: G to enter/exit grab mode; WASD movement; Q/E vertical; Shift for speed

### Shader System

Shaders use Slang (`.slang` files). The CMake build sets `DEFAULT_SHADER_INCLUDE_PATHS` to `src/` so shaders can import each other. Shader types:
- `.cs.slang` — Compute shaders
- `.3d.slang` — Graphics shaders (vertex/fragment/mesh)

Shared shader data structures are in `src/Scene/TetrahedronScene.slang`.

### Key Dependencies

- **Rose Engine** (`extern/Rose`) — Custom Vulkan abstraction layer (submodule). Provides window management, pipeline caching, radix sort, prefix sum, ImGui integration. **Must be linked first** to use its frozen Vulkan headers.
- **Eigen3** — Linear algebra (PCA-based camera normalization in benchmark)
- **GLM** — Math library (configured with `GLM_FORCE_XYZW_ONLY`)
- **Slang** — Shader compiler (runtime shader compilation via RoseLib's `ShaderModule`)

### Compile Definitions

`WIN32_LEAN_AND_MEAN`, `_USE_MATH_DEFINES`, `GLM_FORCE_XYZW_ONLY`, `IMGUI_DEFINE_MATH_OPERATORS`, `VULKAN_HPP_NO_STRUCT_CONSTRUCTORS`

---
> Source: [Shmaug/DelTetRenderer](https://github.com/Shmaug/DelTetRenderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
