## prior-portfolio-shared-infra

> Cross-paper infrastructure conventions


# Cross-paper infrastructure

The portfolio uses a shared infrastructure pattern. Code that is cross-paper lives in `ethical-alpha-audit-shared-repro-core`. Code that is paper-specific lives in the paper's own repo, in modules named after the paper (e.g. `p4_replay/` for Paper 4, `p5_*/` for Paper 5).

When writing or reviewing code:

- **Generators, observation models, SCM, governance engines** are cross-paper concerns and live in `src/` (within each paper repo, mirrored from shared-repro-core for self-containedness).
- **Paper-specific replay logic, paper-specific outcome computation** lives in `p<N>_<topic>/`.
- **Engine modules** (`engine/`) hold the operational audit engine. Paper 4 has the canonical structure; other papers should follow it.
- **Tests** live in `tests/` at the repo root, with one test module per source module being tested.
- **Configs** live in `config/` (parameter YAMLs), `thresholds/` (threshold profiles), and `analysis_plan.yaml` at the repo root.
- **Inputs to experiments** live in `inputs/experiment_pack/`. The user-facing manuscript lives in `inputs/manuscript.docx`.

If you find a piece of code that should be cross-paper but is duplicated across paper repos, surface it: `flag for hoist to shared-repro-core`. Do not auto-hoist; cross-paper code changes affect every consumer and need user approval.

---
> Source: [ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification](https://github.com/ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
