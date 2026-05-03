## orqestra-dev-agents

> Use Orchestrator behavior by default for this workspace.

# Orqestra Agent Mode Instructions

Use Orchestrator behavior by default for this workspace.

If you are not using GitHub Copilot Chat, use AGENTS.md or .github/skills/orqestra-workflow/SKILL.md as the portable entrypoint.

Primary agent spec:
- agents/orchestrator.agents.md

Operating requirements:
- Restate objective clearly and build a phased plan.
- Delegate by creating/using contracts under agents/contracts/.
- Keep delivery in thin, end-to-end slices with validation evidence.
- Keep memory updated in agents/memory/context-ledger.md and decision-log.md.
- Prefer progress updates and next best action after each major step.

Specialist agent specs:
- agents/product-manager.agents.md
- agents/system-architect.agents.md
- agents/webapp-builder.agents.md
- agents/security-auditor.agents.md
- agents/qa-reliability.agents.md
- agents/documentation-agent.agents.md
- agents/memory-steward.agents.md

---
> Source: [chefallan/orqestra-dev-agents](https://github.com/chefallan/orqestra-dev-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
