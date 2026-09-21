## three-rs-wasm

> - Prefer the official WebGPU example when an equivalent WebGL scene exists;

# Reproduction acceptance

- Prefer the official WebGPU example when an equivalent WebGL scene exists;
  list only the WebGPU port. For WebGL-only scenes, document backend-related
  image tolerance without relaxing behavior or performance requirements.
  Follow docs/example-policy.md for selection and comparison.

- Treat performance and execution architecture as part of Three.js compatibility.
  Matching appearance or successful import alone does not establish reproduction.
- Do not replace GPU per-vertex work with CPU/Wasm per-vertex work, stream unchanged
  geometry every frame, or substitute CPU merging for dynamic GPU batching and call
  it equivalent. CPU evaluation for explicit queries/test oracles is separate.
- Keep materially inferior fallbacks/prototypes explicitly unsupported for parity.
  Do not promote their examples to reproduced status or conceal limitations.
- Validate appearance/behavior and steady-state resource costs. Enforce geometry
  residency/upload regressions in tests. Claim timing parity only with comparable
  original Three.js workloads and recorded measurements, not a display-capped FPS.
- Follow docs/performance-parity.md when implementing or reviewing example ports.

---
> Source: [takahirox/three-rs-wasm](https://github.com/takahirox/three-rs-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
