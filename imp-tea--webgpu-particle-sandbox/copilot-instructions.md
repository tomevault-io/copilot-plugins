## webgpu-particle-sandbox

> This is a browser-based 2D WebGPU particle sandbox built with TypeScript, Vite, and WGSL. The simulation is particle-first: bodies are clusters of particles with explicit neighbor bonds, body-local velocity damping, shape-memory correction, wall collision, and cross-body contact projection. Rendering draws instanced particle sprites from the active GPU particle buffer.

# AGENTS.md

## Project Overview

This is a browser-based 2D WebGPU particle sandbox built with TypeScript, Vite, and WGSL. The simulation is particle-first: bodies are clusters of particles with explicit neighbor bonds, body-local velocity damping, shape-memory correction, wall collision, and cross-body contact projection. Rendering draws instanced particle sprites from the active GPU particle buffer.

Use `npm.cmd run build` on Windows PowerShell to type-check and build. Use `npm.cmd run dev` to run the Vite development server.

## Important Files

- `src/main.ts`: app bootstrap, UI binding, startup scene, render loop, simulation pass orchestration, CPU-side body/joint/motor setup.
- `src/input.ts`: pointer input and modifier-click routing for body placement, deletion, dynamic joints, and normal dragging.
- `src/config.ts`: shared constants, byte sizes, limits, and default simulation settings.
- `src/gpu/buffers.ts`: GPU buffer allocation and CPU-side particle/body/bond/rest-shape/joint data creation.
- `src/gpu/pipelines.ts`: WebGPU pipeline, bind group layout, and bind group creation.
- `src/shaders/simulate.wgsl`: integration, gravity, pointer drag, motors, same-body bonds, shape memory, wall collision, floor tangential deadband.
- `src/shaders/solveJoints.wgsl`: soft pin-joint projection between bodies.
- `src/shaders/solveContacts.wgsl`: spatial-grid contact projection between non-jointed bodies.
- `src/shaders/render.wgsl`: particle sprite rendering, including packed per-particle SVG colors.
- `test.svg`: default SVG asset used when spawning an SVG body without selecting a file.

## Current Simulation Pipeline

The particle state uses two ping-pong storage buffers. Each compute pass reads from the active buffer, writes to the inactive buffer, then `ParticleBuffers.swap()` flips the active index.

Per frame:

1. Write `SimParams`.
2. For each substep:
   - `simulate`: integrate forces, motors, damping, bonds, shape memory, and walls.
   - `solveJoints`: project configured body joints for `jointIterations`.
3. If more than one body exists, run contact iterations:
   - rebuild the spatial grid.
   - `solveContacts`: project overlapping particles unless their bodies are directly jointed.
4. Render from the active particle buffer.

CPU particle snapshots are used for picking, vector rendering, drag previews, Ctrl/Meta placement, Ctrl/Meta right-click deletion, and Shift-click joint creation. Keep new interaction features tolerant of snapshot latency; do not add per-frame GPU readbacks beyond the existing throttled snapshot/picking paths.

## Recent Rigidbody-Style Additions

- The startup scene now loads a simple car: a rectangular chassis plus two circular wheel bodies.
- `A` and `D` drive the wheel motors by writing per-body motor target angular velocity and strength into the body buffer.
- Selecting a motorized body shows a `Motor strength` slider. The default and maximum motor strength is `500`.
- Startup car wheels spawn with maximum friction (`1`) while the chassis keeps the normal scene friction.
- Joints are stored in a fixed-size joint buffer (`MAX_JOINTS`, `JOINT_STRIDE_BYTES`) and currently implement soft pin-style attachment between chassis and wheels.
- Joint-connected bodies intentionally do not collide with each other in `solveContacts.wgsl`.
- The default settings currently emphasize lower-energy behavior: substeps `4`, elasticity `2000`, wall bounce `0`, drag strength `5000`, radius `8`.

## Body Kinds, Static Bodies, and Deletion

- `Particle.flags` uses the low 16 bits for body id. Higher bits encode particle kind (`soft`, `static`, `rope`). Keep the body-id mask behavior intact for picking, contacts, rendering, and joints.
- The body buffer is still `32` bytes. Offset `28` now stores body kind flags, replacing the old padding slot.
- `mass = 0.0` means static / infinite mass. Simulation, contacts, and joints must use inverse-mass helpers instead of directly dividing by `mass`.
- Static bodies are a spawn-time option. Existing body kind is not rewritten from the UI because that would require updating particle flags and masses in both ping-pong particle buffers.
- Deletion is a soft delete: particle ranges, bonds, rest-shape data, and body metadata are zeroed, but particle/body buffers are not compacted and body ids are not reused. `settings.particleCount` remains a high-water mark while live UI stats are computed from `bodyRenderInfos`.

## Rope / Chain Notes

- The body selector includes a `Rope` option. Ropes can be created with the normal Add body button or by Ctrl/Meta-clicking the canvas once for the start and again for the end.
- Rope controls include length multiplier, density, and pinned start/end endpoints. Pinned rope endpoints are represented as static particles with `mass = 0`.
- Ropes use `ParticleBuffers.addRope`, not the soft-body shape sampler. They write normal particle/body/bond/rest-shape buffers and stay compatible with ping-pong particle buffers.
- Rope bonds use adjacent stretch bonds plus weaker two-particle bend bonds within the existing `BOND_SLOT_COUNT = 8` layout.
- Rope rest-shape weights are `0`, so shape memory does not try to preserve a rigid original silhouette. Keep body `softBodyStrength` nonzero if bond relaxation should run.
- In `Vectors` render mode, ropes render as open polylines. Non-SVG soft/static bodies render as filled perimeter polygons, and SVG bodies use the bitmap warp path.

## Canvas Interaction Notes

- Ctrl/Meta + left click places the selected body. For ropes, the first click starts a pending rope and the second click sets the end. Escape cancels pending placement.
- Ctrl/Meta + right click deletes the body nearest the latest CPU snapshot pick.
- Shift + left click on two different bodies creates a soft pin joint between the picked particles. Joint anchors use CPU-side rest positions stored with each `BodyRenderInfo`.
- The placement and joint preview bands are DOM overlays, not simulation geometry.

## SVG Body / Vector Rendering Notes

- The body selector includes an `SVG` option. If no file is selected, the app loads `test.svg`.
- SVG bodies are rasterized on an offscreen canvas, sampled by alpha into boundary and interior particles, and scaled by the normal `Size` slider. The current spawn-size maximum is `480`.
- SVG particle colors are sampled from the rasterized SVG and packed into the high bit of `materialId`; `render.wgsl` decodes packed RGB for particle rendering. Simulation shaders should continue treating `materialId` as opaque.
- SVG bodies use `ParticleBuffers.addSampledBody`, which still writes the same particle/body/bond/rest-shape buffers as built-in shapes. Keep this path compatible with ping-pong particle buffers.
- In `Vectors` render mode, SVG bodies render via a triangle-mesh warp of the original rasterized SVG image using the latest CPU particle snapshot. Non-SVG bodies still render as filled perimeter polygons.
- The SVG vector warp is bitmap-based, not true path deformation; expect possible triangle seams or stretching at low particle counts.

## Solver Bias / Drift Notes

Recent drift mitigation work added low-cost solver symmetrization:

- Shape-memory and joint body-transform sampling use phase-shifted, mirrored sample indices via `solverPhase`.
- Same-body bond relaxation alternates bond slot order based on `solverPhase`.
- `solverPhase` changes once per frame. Avoid per-substep uniform rewrites unless you also account for WebGPU queue/uniform ordering carefully.
- A global velocity deadband was tried and removed because it could freeze motion at small substep `dt`.
- A floor-only tangential deadband remains: tiny `velocity.x` is zeroed only when a particle is clamped against the bottom wall.
- The joint solver also applies floor-tangent deadband after using the actually applied post-clamp correction for velocity feedback. This avoids injecting velocity from correction that was clipped by world bounds.

## Data Layout Cautions

- Keep WGSL struct layouts and TypeScript byte sizes aligned. `SIM_PARAMS_BYTES` is currently `128` to satisfy uniform alignment after adding `solverPhase`.
- The body buffer is still `32` bytes. Offsets `8` and `12` store motor target angular velocity and motor strength; offsets `16`, `20`, and `24` store soft-body strength, viscosity, and friction; offset `28` stores body kind flags.
- The particle buffer is still `32` bytes. Packed SVG colors are encoded in `materialId` with high bit `0x80000000`; particle kind is encoded in `flags` above the low body-id bits. Do not change particle struct layouts without updating every WGSL `Particle` definition.
- Contact bind groups include the joint buffer at binding `7`; joint solve bind groups include it at binding `5`.
- If adding fields to GPU structs, update all shaders that define the same struct shape, not only the shader that uses the new field.

## Development Guidance

- Prefer small WGSL changes followed by `npm.cmd run build` and a browser reload; many WebGPU layout problems only appear at runtime.
- Keep new simulation passes compatible with ping-pong particle buffers.
- Avoid CPU readbacks in the frame loop except for existing debug/picking/snapshot paths.
- Be careful with "sleep" or deadband logic. Applying it globally can suppress gravity or motors; prefer contact-specific thresholds unless a body-level sleep system exists.

---
> Source: [imp-tea/webgpu-particle-sandbox](https://github.com/imp-tea/webgpu-particle-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
