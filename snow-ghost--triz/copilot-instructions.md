## triz

> Maintain a portable, evidence-first TRIZ Agent Skill for software engineering. Treat skills/triz as the single source of truth.

# Repository Guidance

## Purpose

Maintain a portable, evidence-first TRIZ Agent Skill for software engineering. Treat skills/triz as the single source of truth.

## Invariants

- Keep canonical frontmatter limited to name and description.
- Keep the skill name and directory name equal to triz.
- Keep SKILL.md below 500 lines.
- Keep product-specific fields out of canonical SKILL.md.
- Point all plugin manifests to ./skills/.
- Do not add a contradiction-matrix lookup without complete data, attribution, licensing review, and tests.
- Mark software mappings as heuristics rather than classical TRIZ doctrine.
- Preserve the gate that skips TRIZ for routine debugging, measurement, and direct standard patterns.
- Preserve engineering specificity, falsifiable verification, and an honest residual trade-off.

## Validation

Run:

~~~
python3 -m unittest discover -s tests -v
python3 evals/run_eval.py --dry-run
python3 evals/heldout/validate_registry.py
~~~

When available, also run the host's Agent Skills validator and each relevant plugin validator.

## Evaluation Changes

- Add or update a development eval case when changing routing, a contradiction template, a resolution route, or output requirements.
- Compare the same model and harness under baseline and TRIZ conditions.
- Keep raw outputs and blind condition labels until judging finishes.
- Do not claim a quality lift from a smoke test, a non-blind review, or an unreplicated sample.
- Inspect regressions for framework theater: TRIZ vocabulary without better causal, implementation, or verification detail.
- Keep development cases separate from `evals/heldout/registry.json`.
- Do not change `skills/triz`, the frozen registry, rubric, or verifier after a held-out run begins.
- Freeze a held-out registry only after base, oracle, license, overlap, and reviewer checks pass.

## Documentation

- Keep docs/research.md factual and source-linked.
- Separate implemented behavior, measured results, hypotheses, and future work.
- Update all plugin versions together for a release.
- Keep preregistered benchmark claims within the models, harnesses, commits, and cases actually measured.

---
> Source: [snow-ghost/triz](https://github.com/snow-ghost/triz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
