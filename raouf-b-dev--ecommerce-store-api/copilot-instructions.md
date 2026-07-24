## ecommerce-store-api

> Canonical source of truth: [AGENT.md](AGENT.md).

# Claude Adapter Policy

Canonical source of truth: [AGENT.md](AGENT.md).

## Rules

1. Do not violate DDD and Hexagonal boundaries.
2. Keep controllers thin and use-case driven.
3. Use ACL gateways for cross-context data access.
4. Run verification appropriate to change risk.
5. Escalate for high-risk ambiguity.

## Workflow

Follow: Intake -> Plan -> Execute -> Verify -> Handoff.

## Skills

Auto-discover skills from:

- `.claude/skills/*/SKILL.md`
- `.agents/skills/*/SKILL.md`

Prefer skill-guided execution when scope matches.

Primary generation skill:

- `.claude/skills/module-conventions-generator/SKILL.md`

## Context Acceleration

Before starting any task, read `.agents/PROJECT-CONTEXT.md` for a compact project snapshot. This saves significant context-gathering time.

---
> Source: [raouf-b-dev/ecommerce-store-api](https://github.com/raouf-b-dev/ecommerce-store-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
