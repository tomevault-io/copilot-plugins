## threejs-conference

> TSL / WebGPU shader and post conventions for Threejs-Punk


# TSL & WebGPU in this repo

Read [AGENTS.md](../../AGENTS.md) before adding effects. Rain: [docs/techniques/collision-rain.md](../../docs/techniques/collision-rain.md). README **Source map** lists entry functions.

- Use `three/webgpu` + `three/tsl`; node materials (`MeshStandardNodeMaterial`, etc.), not raw GLSL files.
- Reusable graphs live in `src/tsl/`; wire from world/post, do not duplicate drop/ripple logic.
- Rain collision: height RT in `createCollisionHeight.js` + compute in `createCollisionRain.js`; update hide list if new fullscreen layers exist.
- Ground wet = ripples + planar reflection RT; car wet = `surfaceRain.js` on UV1.
- Tune cost via `src/platform/performanceProfile.js` (half-res, frame skip, instance counts).
- Post changes: integrate through `createPostProcessing` / `rebuildSteadyOutput`; test Safari constraints (DoF off).

---
> Source: [ektogamat/threejs-conference](https://github.com/ektogamat/threejs-conference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
