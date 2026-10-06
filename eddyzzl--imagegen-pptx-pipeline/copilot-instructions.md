## imagegen-pptx-pipeline

> This repository packages the installable `imagegen-pptx-pipeline` skill.

# AGENTS.md

This repository packages the installable `imagegen-pptx-pipeline` skill.

## Read First

- `imagegen-pptx-pipeline/SKILL.md` — runtime workflow and hard invariants.
- `CONTRIBUTING.md` — repository layout, contribution rules, and PR checks.
- `README.md` — installation, capabilities, examples, and local Codex sync.

## Source Boundaries

- Keep runtime instructions in `imagegen-pptx-pipeline/SKILL.md` and its
  `references/`; do not duplicate the pipeline contract in root instruction files.
- Keep repository documentation, CI, tests, and contributor tooling at the root.
- Encode blocking rules as schemas, deterministic checks, and tests rather than
  prose-only requirements.
- Do not add private templates, user data, generated customer decks, credentials,
  confidential materials, or personal absolute paths.

## Validation

Use the repository validation entrypoint:

```bash
scripts/check --quick
scripts/check --full
scripts/check --privacy
```

`--quick` compiles changed Python files and runs smoke tests. Before publishing,
run `--full` and `--privacy`; review every broad privacy match rather than
treating the raw match count as a failure.

## Change Workflow

- When a runtime invariant changes, update the skill, relevant references,
  deterministic gates, and smoke tests together.
- Keep the inner `imagegen-pptx-pipeline/` directory independently installable.
- Prefer `scripts/check --quick` while iterating, then run the full commands above
  before a PR or release.

---
> Source: [eddyzzl/imagegen-pptx-pipeline](https://github.com/eddyzzl/imagegen-pptx-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
