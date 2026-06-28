## specrail

> This repository defines reusable repository workflow contracts. Keep changes

# AGENTS.md

This repository defines reusable repository workflow contracts. Keep changes
small, explicit, and verifiable.

## Agent Entry

- Treat SpecRail as an agent-facing workflow contract, not a human project
  management guide.
- Read `AGENT_USAGE.md` before creating issues, specs, PRs, reviews, or
  handoffs.
- Use `PLAN.md` for current direction, known limits, and roadmap.
- When the user writes Chinese or the selected locale is `zh-CN`, write
  human-facing issue/spec/PR/handoff text in Chinese while keeping stable IDs,
  paths, commands, and JSON keys in English.

## Rules

- Search before adding a new workflow, schema, template, check, or policy.
- Prefer deterministic checks before LLM or agent automation.
- Do not grant agents final approval, merge, or security-disclosure authority.
- Keep templates generic; repository-specific behavior belongs in examples or
  consumer overlays.
- Preserve the dry-run default for all GitHub automation.

## Validation

Run before completion after changing workflow assets:

```sh
python3 checks/check_workflow.py --repo .
```

---
> Source: [majiayu000/specrail](https://github.com/majiayu000/specrail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
