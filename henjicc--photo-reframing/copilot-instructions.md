## photo-reframing

> - Focus renderer work on `src/renderer/three` and keep UI changes narrowly scoped unless the requested behavior requires more.

# AGENTS.md

## Project Notes

- Focus renderer work on `src/renderer/three` and keep UI changes narrowly scoped unless the requested behavior requires more.
- The Gaussian viewer uses Spark (`@sparkjsdev/spark`) through `SparkRenderer` and `SplatMesh`; when enabling `covSplats`, also enable `extSplats`.
- Rendering settings should update live in the current viewer. Inference settings may remain next-upload only.

## Verification Discipline

- Do not run expensive builds, smoke tests, packaging, or broad test suites by default.
- Prefer the smallest useful check for the file touched, and skip formal verification when the change is obvious and low-risk.
- Run `npm run build` only when changing bundling, dependencies, renderer integration, TypeScript boundaries, or other areas where a compile-time/runtime integration failure is likely.
- Ask before running long Electron smoke tests or packaging flows unless the user explicitly requests them.

---
> Source: [henjicc/Photo-Reframing](https://github.com/henjicc/Photo-Reframing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
