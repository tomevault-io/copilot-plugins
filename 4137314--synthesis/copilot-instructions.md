## synthesis

> This is a Lean 4 library for formal engineering technology descriptions lowered

# Working on synthesis

## Scope and architecture

This is a Lean 4 library for formal engineering technology descriptions lowered
into an open, backend-neutral engineering IR. The package is `synthesis`, namespace `Synthesis`.
Read README.md and docs/architecture.md before changing the architecture.
Implement no VHDL, GDSII, Python, simulator, fabrication or other target backend.
External consumers own target support and must reject unsupported operations.

Dependency direction: Core → Physics/IR; Logic and Systems stand independently;
IR + Logic → Semantics → Design; Design + IR + Semantics → Frontend → Examples.
Interop depends on IR and Semantics and defines external pipeline contracts.
The umbrella Synthesis.lean exports the kernel. Domain packages depend on the kernel,
never the reverse. Tests may depend on everything; library modules never import tests.
Read docs/domain-development.md and docs/models.md before introducing domain semantics.
All repository code, documentation, comments and diagnostics must be in English.
Use explicit variables (`autoImplicit false`) in new model modules.
Do not import Domains or Bridges through the kernel umbrella.

## Workflow

- Inspect existing files and git diff first; preserve unrelated user changes.
- Use `nix develop` for tools; `lean-toolchain` and `flake.lock` pin versions.
- Use `bash scripts/check.sh` before completion, including module-closure checks and Tests/Audit.lean.
  The assumption audit permits only propext, Classical.choice and Quot.sound.
  Report checks actually run and blockers.
- Keep changes bounded and reviewable. Update docs and examples with API changes.
- Add regression checks for new invariants and rejection paths, not just happy paths.
- Use a short plan for cross-module changes; record lasting decisions in docs/adr.
- Do not add dependencies without a concrete requirement; pin any introduced dependency.
  Mathlib is authorized and pinned. Prefer focused imports. doc-gen4 belongs in docbuild.
  Update both Lake manifests together when changing shared dependencies.
- Run `bash scripts/docs.sh` for public API/documentation changes before publishing.
  Pages deployment must depend on successful proof checks and verified generated pages.
- Do not commit, publish, deploy or contact external services unless requested.
- Agents must never credit themselves as commit authors or co-authors. Never add a
  `Co-authored-by` trailer, agent attribution, or an AI signature to commits or PRs.
  Use the user's existing Git identity; do not replace it with an agent identity.

## Formal boundary

Never use `sorry`, `admit`, new `axiom` declarations, or `unsafe` in library code.
Do not replace kernel-checked proofs with unchecked assertions or native evaluation.
Distinguish structural validation, dimensional typing and proven physical semantics.
Each theorem must state its assumptions. A named operation is not a physical model.
Document every new model in docs/models.md with its assumptions and named theorems.
Physical law definitions and hypotheses are distinct from proved logical consequences.
IR schema 3 stores open symbolic terms and exact literals: never hide varying
coefficients behind an identical definition identity. Update the schema ADR when
changing this contract. Unknown extension data requires explicit coverage or rejection.
Preserve dimensions and domain identity; model transducers explicitly.
Quantum ports are resources, not ordinary clonable classical wires.
Engineering certificates require Semantics.Verified for a specific model/requirement,
including feasibility. Frontend.Certified only certifies a graph predicate.
Unknown component and coupling interpretations must return none, never True.
Prove operating-envelope assumptions and explicitly state unproved applicability,
realizability, liveness and uncertainty obligations. Never claim industrial certification.
Compiler guarantees must distinguish equivalence, refinement and explicit lossy projection.
Refinement certificates require feasibility; merely valid output is insufficient.
Changing AST semantics requires a schema-version decision and an ADR.
Do not silently discard unsupported constructs or claim backend realizability.

## Definition of done

Public API documented, relevant positive/negative tests pass, examples compile,
no proof holes, no accidental backend logic, and remaining limitations reported.

---
> Source: [4137314/synthesis](https://github.com/4137314/synthesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
