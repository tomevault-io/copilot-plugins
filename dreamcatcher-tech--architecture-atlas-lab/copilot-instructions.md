## architecture-atlas-lab

> Architecture Atlas Lab is an intentionally separate visual experiment.

# Agent instructions

Architecture Atlas Lab is an intentionally separate visual experiment.

- Keep `src/model/` deterministic and independent of DOM, Canvas, SVG, or Three.js.
- All views must derive from the same solver result and stable candidate IDs.
- Preserve the distinction between hard-constraint elimination and objective selection.
- Pure hard-constraint order presets must converge to the same feasible set.
- Every visual aggregate must reconcile exactly to its underlying candidate count.
- A tree/DAG is a projection; do not make rendered layout the canonical model.
- State bounded completeness and source provenance visibly.
- Do not modify the source synthesis, Atlas, Fundamentals, or notes repositories from this repository.
- Run `npm run check` before commit and visually inspect the deployed Pages site at desktop, tablet, and phone widths.

---
> Source: [dreamcatcher-tech/architecture-atlas-lab](https://github.com/dreamcatcher-tech/architecture-atlas-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
