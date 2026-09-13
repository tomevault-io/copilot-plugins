## paper-collage-video

> These instructions apply to the entire repository.

# Repository Agent Guidance

These instructions apply to the entire repository.

## Read the Evolution Policy

Before planning or implementing changes to the Skill, project contracts, renderer,
provider workflow, templates, validation, or quality gates, read
[`docs/skill-evolution-principles.md`](docs/skill-evolution-principles.md).

## Prefer the Right Architecture

- Start from the visual language and production behavior the output needs. Do not
  treat the current schema or renderer as an immutable boundary.
- If the current abstraction cannot express a reusable behavior cleanly, change
  the abstraction. Implement the schema, runtime, authoring path, validation,
  proof/fingerprint logic, quality review, tests, documentation, and packaged
  plugin as one coherent vertical slice.
- Do not ship project-specific branches, large piles of opacity-controlled image
  nodes, baked-in dynamic UI, duplicated timing logic, or manual repair steps as
  the production design. A workaround may prove feasibility, but it is not proof
  that the Skill professionally supports the behavior.
- In design and review, distinguish clearly between existing first-class support,
  a temporary feasibility workaround, and an architectural gap that should be
  fixed before production.

## Compatibility Policy

- Maintain output-capability compatibility, not legacy-project compatibility.
- Breaking changes to old project files, schemas, assets, directories, state,
  caches, and internal APIs are allowed. Old projects do not need to resume or
  load in the latest system.
- Do not add migration loaders, dual-schema parsers, deprecated fields, adapters,
  or version-conditioned renderer branches unless the user explicitly requests
  legacy compatibility.
- A superseded path should normally be removed rather than kept beside its
  replacement.
- The latest system must still be capable of regenerating the useful classes of
  final video that earlier systems produced. If an old production must be revised,
  rebuild it as a new project under the latest contract.

## Cost-Efficiency Policy

- Preserve the approved quality bar while minimizing paid/provider calls, render
  work, and avoidable regeneration.
- Batch related states from one identity or prop into a registered 2x2, 3x2, or
  similarly dense sheet when the provider can produce it reliably; split and key
  it deterministically while preserving one canvas for every cell.
- Do not batch unrelated identities merely to fill a grid. If one reviewed cell
  fails, first try deterministic local reprocessing. A quota-consuming targeted
  repair must edit a mask inside the complete original sheet so every accepted
  cell remains provider context and can be proven unchanged. If that is not
  reliable, regenerate the complete sheet. Never generate a replacement cell in
  isolation for a multi-state family.
- Provenance and reports must distinguish provider calls from deterministic local
  derivatives and expose the calls actually avoided.

## Verification Policy

- Test the latest contract and representative output capabilities. Do not retain
  legacy fixtures merely to prove that obsolete project data still loads.
- A new reusable visual behavior needs deterministic validation and visual proof,
  not only a successful render from one hand-authored project.
- When Skill/runtime files change, run the relevant tests and checks, then run
  `npm run plugin:sync` and validate the packaged copy so source and plugin do not
  drift.

---
> Source: [cyberlesterr/paper-collage-video](https://github.com/cyberlesterr/paper-collage-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
