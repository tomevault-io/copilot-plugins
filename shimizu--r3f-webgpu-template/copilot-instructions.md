## r3f-webgpu-template

> **Rogue Hunter** (Project Name: **r3f-webgpu-template**) is a specialized GIS visualization framework designed as a "Diorama-style" miniature world. It integrates high-fidelity environmental elements (sky, water, terrain, weather) with real-time geospatial data (GeoJSON, moving entities) using **WebGPU** and **Three Shading Language (TSL)**.

# GEMINI.md - Project Context

## Project Overview
**Rogue Hunter** (Project Name: **r3f-webgpu-template**) is a specialized GIS visualization framework designed as a "Diorama-style" miniature world. It integrates high-fidelity environmental elements (sky, water, terrain, weather) with real-time geospatial data (GeoJSON, moving entities) using **WebGPU** and **Three Shading Language (TSL)**.

The project shifts away from traditional flat map SDKs toward a "boxed world" experience where GIS data is treated as an exhibit on a physical-looking diorama stage.

### Key Concepts
- **Diorama as Host, GIS as Guest**: The look and feel are driven by the diorama environment (lighting, post-effects, sky). GIS data is an interpretive layer within this stage.
- **GPU-First Architecture**: While the CPU handles data fetching and orchestration, the GPU (via TSL) handles projection (Lon/Lat to XY), interpolation, particle advection, terrain collision, and rendering.
- **Miniature Scale**: The world is scaled down to fit on a "tabletop" (e.g., `WORLD_VIEW` scale), using `MapControls` for exploration.

## Core Technologies
- **Frontend Framework**: React 19 + React Three Fiber (R3F)
- **Renderer**: Three.js `WebGPURenderer`
- **Shader Language**: Three Shading Language (TSL) / WGSL
- **Build Tool**: Vite 8
- **GIS Utilities**: `geotiff` (for DEM), `earcut` (polygon triangulation)
- **UI**: `leva` (for live parameter tuning)

## Architecture & Directory Structure
- `src/App.jsx`: Entry point; initializes the `WebGPURenderer` and R3F `Canvas`.
- `src/Scene.jsx`: The composition root; assembles lighting, environment, and all visual layers.
- `src/layers/`: React components representing individual visual elements.
    - **Environment**: `SkyLayer` (static gradient + fBM clouds, tabletop tone), `GridLayer` (diorama floor), `StageLayer`.
    - **Simulation**: `WaterBoxLayer`, `WaterBlobLayer`, `WaterOceanLayer` (TSL-based water simulations).
    - **GIS**: `GeojsonLayer` (vector maps), `MovingEntitiesLayer` (interpolated GPU particles).
    - **Weather/Terrain**: `TerrainLayer` (GeoTIFF-based 3D mesh), `CloudLayer` (TSL raymarched volumetric clouds: cumulus / stratus / cirrus), `RainLayer` (GPU particles with collision).
- `src/compute/`: TSL-based compute shader definitions.
    - `createInterpolationPass.js`: Temporal interpolation + projection for moving entities (dateline-aware).
    - `runRainCompute.js`: Physics, wind field (FBM), and terrain collision logic.
    - `disposeStorageAttributes.js`: GPU buffer release helper called from each pass's `destroy(renderer)`.
- `src/gis/`: Logic for projections (`projectionGPU.js`), view settings (`views.js`), and coordinate transforms.
- `docs/`: Technical specifications for the particle system, projection unification, and WebGPU quality.

## Building and Running
- **Development**: `npm run dev`
- **Linting**: `npm run lint`
- **Build**: `npm run build`
- **Preview**: `npm run preview`

## Development Conventions
- **TSL Preference**: Favor TSL (`three/tsl`) over raw WGSL or GLSL for shader logic to maintain compatibility with the Three.js ecosystem.
- **Surgical GPU Updates**: CPU should only pack data into `StorageBufferAttribute`. All frame-by-frame updates (positioning, physics) must occur in compute shaders.
- **Layered Composition**: Each feature should be a self-contained `*Layer.jsx` component.
- **Project Scope**: Refer to `plan.md` for current development phases (Phase A-E) and `task.md` for specific to-dos.
- **No Map SDKs**: Avoid importing large map libraries (Mapbox, Leaflet). Implement required GIS logic from scratch or using lightweight utilities.

## Current Focus (Phase B/C)
- Phase A (spatial integration of terrain + GIS via the `<Coordinate>` projection context) is complete.
- Terrain-linked GIS particles (entities following terrain height / sea surface).
- Implementing GPU-based trails for moving entities.

---
> Source: [shimizu/r3f-webgpu-template](https://github.com/shimizu/r3f-webgpu-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
