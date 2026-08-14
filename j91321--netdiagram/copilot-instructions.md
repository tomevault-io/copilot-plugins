## netdiagram

> Static Vue 3 (JavaScript) web app for authoring network deployment diagrams and

# netdiagram — agent conventions

Static Vue 3 (JavaScript) web app for authoring network deployment diagrams and
exporting clean, id-addressable SVG. No backend.

## Read before coding
1. `ARCHITECTURE.md` — decisions, tech choices, source tree.
2. `docs/DATA_MODEL.md` — diagram JSON schema + icon manifest schema.
3. `docs/SVG_EXPORT_SPEC.md` — **normative** export contract for downstream apps.
4. `docs/BUILD_PLAN.md` — milestones, tasks, acceptance criteria.

## Hard rules
- **JavaScript only** — no TypeScript files, no `lang="ts"`. This is a product
  owner requirement, not a preference to negotiate.
- JSON document is the source of truth; SVG is generated from it by
  `src/services/svgExport.js`, never scraped from the canvas DOM.
- Exported SVG: no `foreignObject`, no scripts, no raster images, plain `<text>`
  labels, every entity a `<g>` with user id + `data-type`. Any structural change
  to the export must update `docs/SVG_EXPORT_SPEC.md` in the same commit.
- Icons live in `public/icons/` + `manifest.json`; adding an icon must never
  require an app rebuild.
- Business logic (export, import, geometry, id utils, icon normalization) goes
  in framework-free modules under `src/services/` and `src/model/`, unit-tested
  with Vitest. Vue components stay thin.
- No new runtime dependencies beyond `vue` + `pinia` without a justification
  added to ARCHITECTURE.md.

## Commands
- `npm run dev` / `build` / `preview`
- `npm test` (Vitest), `npm run lint` (ESLint flat config)
- `npm run icons:check` — validate icon manifest ↔ files

## Definition of done (every change)
`npm test` and `npm run lint` pass; the current milestone's acceptance criteria
in `docs/BUILD_PLAN.md` hold; specs updated if behavior changed.

## Reference material
`examples/sample-export.svg` is a real export — zones with
title/IPv4/IPv6/description labels, iconed hosts with hostname+IP, connections —
alongside the `sample-export.json` it was generated from. That pair is the
worked example to read.

The draw.io export this project was modelled on has been removed from the
repository and its history; do not re-add it.

---
> Source: [j91321/netdiagram](https://github.com/j91321/netdiagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
