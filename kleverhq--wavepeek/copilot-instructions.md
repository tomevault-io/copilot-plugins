## wavepeek

> This directory owns GitHub Actions workflow YAML.

# Workflow Guidance

## Scope

This directory owns GitHub Actions workflow YAML.

## Source of Truth

- Workflow automation: `../../docs/automation.md`
- Quality gates: `../../docs/quality.md`
- Stable local task interface: `../../justfile`
- Workflow helper scripts: `../../tools/`

## Local Guidance

- Validate workflow changes with `just check-actions`.
- Keep inline shell or Python in workflow `run` and `runCmd` blocks to short glue up to 5 logical lines.
- Move longer logic into a separate testable helper under `../../tools/`, then invoke that helper from the workflow.

---
> Source: [kleverhq/wavepeek](https://github.com/kleverhq/wavepeek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
