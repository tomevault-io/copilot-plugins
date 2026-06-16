## weighted-intelligence-nodes

> Run the full WIN committee flow: profile targets, evaluate drafts, and synthesize consensus feedback. Use for end-to-end message testing, variant comparison, and rewrite prioritization.


# WIN Committee

## Overview

Execute the end-to-end committee pipeline for hackathon and MVP messaging validation.

Use this orchestration skill when the user wants full-loop output (`profiles.json`, `committee_matrix.json`, and `summary.md`).

For single-stage tasks, prefer focused skills:
- `win-profile`
- `win-evaluate`
- `win-summary`

## Workflow

1. Validate inputs against the contracts in `references/data-contracts.md`.
2. If required inputs are missing, please interview the user to acquire the necessary inputs before running commands.
3. Run profiling from target public data.
4. Run committee evaluation across experts and content drafts.
5. Synthesize output into a balanced summary with consensus signals.

## Commands

Run full pipeline:

```bash
scripts/run_committee.sh
```

Use the Laura evidence-rich example as input:

```bash
cd /Users/tomago/andrew-tomago/public/weighted-intelligence-nodes
cp examples/targets.laura-modiano.example.json data/input/targets.local.json
```

Run staged pipeline:

```bash
python3 ../../../scripts/mvp_committee.py profile \
  --targets ../../../data/input/targets.local.json \
  --out ../../../data/output/profiles.json

python3 ../../../scripts/mvp_committee.py evaluate \
  --profiles ../../../data/output/profiles.json \
  --content ../../../data/input/content.local.json \
  --committee ../../../config/committee.json \
  --out ../../../data/output/committee_matrix.json

python3 ../../../scripts/mvp_committee.py synthesize \
  --matrix ../../../data/output/committee_matrix.json \
  --out ../../../data/output/summary.md
```

## Tuning Rules

- Add or remove committee experts in `config/committee.json`.
- Adjust score emphasis via `rubric_weights`.
- Define `judging_criteria` labels/descriptions in judge language to improve persona realism.
- Keep weights normalized enough to avoid one expert dominating output.
- Add domain-specific focus keywords per expert to reflect committee specialization.
- Check `committee_matrix.json` `rationale` for empathy and specificity before sharing summary.

## Guardrails

- Use only public or user-provided data.
- Never infer private or sensitive attributes.
- If required inputs are missing, please interview the user to acquire the necessary inputs.
- Keep analysis focused on content quality and audience-fit signals.
- Stick to observable public work and stated preferences only.
- Keep user-specific input in `data/input/*.local.json` (gitignored).
- Keep all file paths repository-relative or skill-relative; avoid host-specific absolute paths.

## References

- `references/workflow.md`
- `references/data-contracts.md`
- `scripts/run_committee.sh`

---
> Source: [andrew-tomago/weighted-intelligence-nodes](https://github.com/andrew-tomago/weighted-intelligence-nodes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
