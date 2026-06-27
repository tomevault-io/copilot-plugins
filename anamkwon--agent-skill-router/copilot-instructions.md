## skill-router

> Use Skill Router to organize large local agent skill libraries into category routers.


# Skill Router

Use this rule when organizing, reviewing, or refreshing many local agent skills.

Run `node scripts/skill-router.mjs scan` first. Read the compact routing review
and inventory before opening individual leaf skill bodies.

Similarity signals such as BM25 score, token overlap, and duplicate
descriptions are only read-priority hints. The agent decides the final category
assignment from workflow force, stop conditions, and intended behavior.

Read a leaf `SKILL.md` only when `routing-review.md` marks it under
`Needs Body Review`, the compact inventory is ambiguous, or a category boundary
would change. Write `routing-plan.json`, then run:

```bash
node scripts/skill-router.mjs apply
node scripts/skill-router.mjs scan
node scripts/skill-router.mjs validate
```

Do not create catch-all categories such as misc, general, or other.

---
> Source: [AnamKwon/agent-skill-router](https://github.com/AnamKwon/agent-skill-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
