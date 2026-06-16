## founder-os

> Multi-agent primitives for Claude Code: consensus, debate, fanout research, skill building, and measured optimization.


# founder-os

Use `founder-os` when one Claude Code pass is not enough and you want to compare multiple agent runs, surface disagreement, or synthesize parallel research.

## Included primitives

- `/stochastic`: poll N agents with the same prompt and aggregate consensus, divergences, and outliers.
- `/model-chat`: run a multi-agent debate room and synthesize the strongest position.
- `/fanout`: send researchers down parallel paths, then merge findings.
- `/skillbuilder`: build higher-quality Claude Code skills by identifying what the skill exploits.
- `/autoresearch`: run measured optimization loops against a numeric metric.

Install:

```bash
claude plugin install rhinehart514/founder-os
```

---
> Source: [rhinehart514/founder-os](https://github.com/rhinehart514/founder-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
