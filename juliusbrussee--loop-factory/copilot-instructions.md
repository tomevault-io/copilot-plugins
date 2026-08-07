## loop-factory

> Loop Factory is a spec-driven orchestration repo. Preserve this boundary:

# Agent Instructions

Loop Factory is a spec-driven orchestration repo. Preserve this boundary:

- Specs decide product intent.
- Agent loops implement, verify, review, and backpropagate facts.
- Do not let automation silently make product decisions.

## Commands

Use local CLI first:

```bash
python3 -m unittest discover
python3 bin/loop-factory doctor
python3 bin/loop-factory scan
```

Use `loop-factory dispatch --stage` to move specs into active state. Use `--execute` only when user asked for live agent CLI execution.

## Repo Rules

- Keep Python dependency-free unless strong reason.
- Keep generated prompts under `factory/prompts/`.
- Keep run metadata under `factory/runs/`.
- Move accepted specs with `loop-factory archive <id> --accepted`.
- Update docs/specs when code changes invalidate them.
- Do not archive specs before review evidence exists.

## Review Stance

Findings first. Prioritize correctness, behavior regressions, missing tests, security, and maintainability.

---
> Source: [JuliusBrussee/Loop-Factory](https://github.com/JuliusBrussee/Loop-Factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
