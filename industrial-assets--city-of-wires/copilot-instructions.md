## city-of-wires

> - **Goal**: Implement real-time volumetric lighting in dense fog using a camera-relative froxel volume with temporal reprojection. Minimize per-chunk work; everything updates on the fly as procedural chunks stream.


# Volumetric Lighting Blueprint

- **Goal**: Implement real-time volumetric lighting in dense fog using a camera-relative froxel volume with temporal reprojection. Minimize per-chunk work; everything updates on the fly as procedural chunks stream.

## Pipeline (per frame)
- `ClusteredLightCulling` (CPU or GPU) populates light indices per froxel cluster.
- `BuildFroxelGrid` defines a camera-relative 3D volume (log-distributed Z) that follows the player.
- `InjectDensity` clears to global fog baseline, then splats chunk-provided local volumes.
- `InjectLights` accumulates light contributions per froxel from clustered lists (shadowed hero lights flagged).
- `RayMarchSingleScattering` marches view rays (~80 steps) with blue-noise jitter, updating scattering/transmittance.
- `TemporalReprojectionDenoise` reprojects with motion vectors, clamps history, blends (alpha ~0.9).
- `CompositeVolumetrics` combines scattering/transmittance into the main lighting/post stack.

## Math & Storage Conventions
- Extinction/transmittance: `T = exp(-∫ σ_t ds)`; update `T` multiplicatively per step.
- In-scattering increment: `ΔL = T * σ_s * Li * phase * Δs`.
- Phase function: Henyey–Greenstein with `g ≈ 0.6–0.8` (forward-biased fog/rain).
- Store per-voxel: extinction `σ_t`, albedo `σ_s/σ_t`, optional phase `g`.

## Vulkan Resources
- 3D images:
  - Froxel light accumulation: `RGBA16F` (or `R11G11B10_UFLOAT`).
  - Froxel density: `R16F`.
  - History volume: `RGBA16F`.
- 2D textures: blue-noise/scrambling (`R8`/`RG8`).
- SSBOs: light list (SoA), cluster light indices, density volume descriptors, optional freelist metadata.
- Per-frame constants: push constants or UBO with camera matrices, jitter, froxel params.
- Descriptor sets:
  - Set 0: per-frame UBO/push, blue-noise.
  - Set 1: froxel images (rw), history volumes, depth, motion vectors.
  - Set 2: light SSBO, cluster list, density volumes, shadow atlas.

## Render Graph Order (minimal viable)
1. `ClusterBuildCS`
   - Inputs: camera frustum, lights SSBO.
   - Output: cluster light index list for froxel grid.
2. `DensityInjectionCS`
   - Clear density volume to base fog constants.
   - Splat local density primitives (boxes/spheres/cones) streamed with chunks.
3. `LightInjectionCS`
   - Accumulate unshadowed lights analytically per froxel.
   - Mark shadowed hero lights for ray-march sampling (budget ≤ 6).
4. `VolumetricRayMarchCS`
   - March view rays through froxel grid (half/quarter res XY, full Z).
   - Use clustered light lists; sample shadow atlas every N steps (≈5).
   - Output: half-res `RGBA16F` scattering + optional transmittance buffer.
5. `TemporalReprojectionCS`
   - Reproject using motion vectors; neighborhood clamp to prevent ghosts.
   - Blend current/history with `α ≈ 0.9`; optional bilateral blur guided by depth/normal.
6. `CompositeVolumetrics`
   - Combine with scene color: `final = scene * Transmittance + Scattering`.
- Use `vkCmdPipelineBarrier2` between passes with `VK_ACCESS_SHADER_READ_BIT`/`WRITE_BIT` and appropriate stages (compute ↔ graphics).

## Performance Notes
- Froxel grid example: `160 × 96 × 160` for 1440p (`near = 0.5m`, `far = 250m`, log Z).
- Workgroups: `(4,4,4)` or `(8,4,4)` for 3D passes; `(8,8)` or `(16,16)` for screen-space passes.
- Early-out when accumulated `T < 0.01`.
- Temporal reprojection crucial for stabilizing low sample counts; jitter with tiled blue noise (256×256).

## Shadow Strategy
- Maintain a combined shadow atlas for hero lights (spot/rect). Update only flagged lights.
- Sampling policy: evaluate shadow every 4–6 march steps and reuse result; apply small PCF (3×3).
- Non-hero lights rely on analytic attenuation/cookies (no shadows).

## Streaming Integration
- When a chunk loads:
  - Append lights to SSBO (ring buffer) and mark volumetric flags/cookies.
  - Append density volumes (shape, transform, base density/albedo/phase, optional LUT id).
  - Optionally add emissive proxies (e.g., neon spheres) with limited radius.
- When a chunk unloads: reclaim indices via freelist; keep buffers persistently mapped; double-buffer if needed.

## Default Parameters (tweak as needed)
- Froxel grid size: `(160, 96, 160)` camera-relative.
- March steps: `80` with jitter.
- Base fog: `σ_t = 0.015 m⁻¹`, `albedo = 0.9`, `g = 0.7`.
- Temporal blend `α = 0.9`, history clamp ±10% luminance.
- Shadowed hero light budget: ≤ 6.
- Scattering buffer: half-resolution `RGBA16F`.

## Visual Accents
- Add local density cones aligned with searchlights for visible shafts.
- Slight chromatic aberration when compositing scattering.
- Apply anamorphic bloom to scattering buffer.
- Augment hero lights with additive screen-space quads gated by depth/normal to boost shafts.

## Debug & Profiling
- Provide froxel slice viewer (XY/XZ/YZ) to inspect density/light injection.
- Heatmap clustered light counts per froxel to verify culling.
- Toggle hero shadows vs unshadowed to measure cost.
- Capture with RGP/Nsight; monitor occupancy, LDS usage, barrier timings.

## Implementation Reminders
- All code under `pcengine` namespace.
- Respect C++20, Vulkan error handling, and RAII conventions established in the renderer.
- Ensure descriptor layouts and pipeline barriers integrate with existing render graph.
- Keep assets ASCII-encoded unless existing data requires otherwise.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/industrial-assets)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/industrial-assets)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
