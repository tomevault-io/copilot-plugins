## architecture-invariants

> Non-negotiable architecture invariants for mockstudio. Read CONTEXT.md, docs/ARCHITECTURE.md, and docs/adr/ for the full why.


# mockstudio architecture invariants

These are the load-bearing rules. Full rationale lives in `CONTEXT.md` (vocabulary), `docs/ARCHITECTURE.md` (shape), and `docs/adr/` (decisions). When in doubt, read those — do not invent new structure.

## One source of truth
- The render is a pure function: `f(SceneDocument, SourceMedia)`. Nothing renders that is not derived from those two.
- Everything the user can change lives in the **Scene Document**. Presets, persistence, sharing, undo, and export all fall out of that one abstraction.

## Module boundaries
- `core/` imports **no** `react`, `three`, or `@react-three/*` (ESLint-enforced). It emits plain-data **Scene Spec**, never Three.js objects.
- Rule of thumb: *a value a formula could get wrong* (trig, clamps, dimensions, UV, sizing) → `core/`; *handing a number or id to a drei component* → `scene/`.
- Layers: `core/` (pure) → `state/` (zustand) → `scene/` (all Three.js) → `ui/` (Tailwind v4 + shadcn/Base UI) → `app/`.

## What is NOT in the document (ADR-0026, superseding ADR-0014)
- The **Source Media** (Source Image or Source Video) is a separate ephemeral input — never in the document, never serialized (not in `.mock.json`, not in share links). Pixels are input; the mapping (Fit, crop) is document.
- Ephemeral UI state is never in the document: `isExporting`, `isDragging`, active panel, object URLs, toasts, pin/collapse/order.

## Interaction & rendering (ADR-0015, ADR-0009)
- One gesture = one undo entry = one save. Drags write via **transient updates** (refs/subscriptions), not React re-renders; bracket continuous gestures with `beginInteraction()` / `endInteraction()`.
- `frameloop="demand"`: render on `invalidate()`. A drag runs a continuous loop; animated grain instead drives a throttled ~24 fps `invalidate()` (issue 11), never `frameloop="always"`. Idle editor renders zero frames. **Export freezes time.**
- DPR clamped to `[1, 2]`; user textures clamped to `MAX_TEXTURE_SIZE` (mipmaps + anisotropy).

## Color & post-processing (ADR-0007, ADR-0021)
- Linear workflow, tone mapping via pmndrs, renderer tone-mapping **off** (no double tone-mapping).
- The tone-map curve is **document-selected** (ADR-0021, reversing ADR-0007's always-ACES): faithful **Neutral** (Khronos PBR) by default so a bare upload is colour-true, **ACES Filmic** opt-in via cinematic Look Presets / the Tone control. It is still the **last** composer stage.
- Composer order: **Blur → Bloom → Chromatic Aberration → Vignette → Grain → Adjustments → Tone map** with 8× MSAA on the composer (ADR-0018; no SMAA, no depth-based DOF). Screen stylization is on the screen **material**, not the composer.

## Safety & language
- All external input (imported json, share links) is untrusted: always `safeParse` + `migrate`, never crash.
- Use `CONTEXT.md` vocabulary exactly (Mockup, Scene Document, Scene Spec, Frame, Screen, Source Media, Source Image, Source Video, Stage, Fit, Cinematic Effect, Look/Camera Preset, Exporter, Core) and avoid the synonyms it lists.

## Definition of Done
- New `core/` behavior covered by behavior-level Vitest tests (red→green history, not bulk).
- `scene/`/`ui/` behavior verified in the browser with captured evidence.
- typecheck + lint + build + test green locally and in CI.

---
> Source: [lenxism/mockstudio](https://github.com/lenxism/mockstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
