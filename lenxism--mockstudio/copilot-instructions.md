## r3f-live-docs

> Consult live docs (pmndrs / context7 MCP) before writing R3F, drei, or post-processing APIs instead of relying on model memory.


# Use live docs for R3F / drei / post-processing

These APIs change often and model memory is stale. Before writing or changing R3F, drei, or post-processing code, confirm the current API against live docs rather than guessing prop names, signatures, or imports.

## Where to look
- **`user-pmndrs` MCP** — verified good for `react-three-fiber`, `drei`, `zustand`. Use `get_page_content` with the right `lib` + page path (paths have no `/docs` prefix, e.g. `/api/hooks`, `/staging/environment`, `/shapes/rounded-box`).
- **`context7` MCP** — for `@react-three/postprocessing` and the raw `postprocessing` lib. The pmndrs `react-postprocessing` index is currently unreliable, so prefer context7 there. Resolve the library id first; max 3 calls per question.

## When to check (not exhaustive)
- Composer / effect props (Bloom, DepthOfField, Vignette, Noise, ChromaticAberration) and custom `Effect` subclasses.
- drei staging/material APIs (`Environment`, `ContactShadows`, `AccumulativeShadows`, `MeshReflectorMaterial`, `MeshTransmissionMaterial`, `shaderMaterial`, `RoundedBox`).
- R3F render-loop control (`frameloop`, `invalidate`, `useFrame` priority) — the basis of demand rendering and export.

## Rule
If unsure whether a prop or signature is current, look it up before writing it. A 30-second doc check beats a plausible-looking API that doesn't exist.

---
> Source: [lenxism/mockstudio](https://github.com/lenxism/mockstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
