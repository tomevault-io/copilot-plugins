## agent-skill-router

> Use `/organize-skills` when the user wants to organize, refresh, or review a

# Skill Router Extension

Use `/organize-skills` when the user wants to organize, refresh, or review a
large local skill library. The command must use the compact inventory first and
read individual leaf `SKILL.md` files only when the routing review marks them as
ambiguous or body-review candidates.

The core script is `scripts/skill-router.mjs`. It reads
`skill-router.config.json` or the `SKILL_ROUTER_*` environment variables.
Similarity fields are read-priority hints; the agent still decides the final
category plan.

---
> Source: [AnamKwon/agent-skill-router](https://github.com/AnamKwon/agent-skill-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
