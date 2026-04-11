## agent-rules-templates-haskell

> <!-- agent-rules@0_3_0 objective=general language=haskell strictness=balanced repo_name=agent-rules_Engineering -->

<!-- agent-rules@0_3_0 objective=general language=haskell strictness=balanced repo_name=agent-rules_Engineering -->

# agent-rules Engineering Gemini Guidance

Provider target: Gemini

This file supplements `AGENTS.md` with provider-specific configuration.
Full engineering guidance (base rules, workflow contract, task recipes,
verification checklist) is defined in `AGENTS.md`.

## Provider Configuration
- Agent directory: `.gemini/agents`
- Workflow agents: `plan-agent`, `coach-agent`, `build-agent`, `debug-agent`

## Workflow Chain
For every code-change request, delegate through agents in this strict order:

1. `plan-agent` — produce plan, then STOP and wait for user approval
2. `coach-agent` — review risks/tests, return GO or NO-GO
3. `build-agent` (implementation) or `debug-agent` (failures) — execute approved work
4. **verify** — run lint, type-check, and test commands
5. **handoff** — summarize changes, evidence, and next steps

Do not skip steps or reorder. Each agent returns results to the parent before the next is invoked.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/climate-x-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/climate-x-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
