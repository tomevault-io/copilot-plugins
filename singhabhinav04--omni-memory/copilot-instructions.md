## omni-memory

> _Standing instructions for AI coding agents in this repo._

# AGENTS.md

_Standing instructions for AI coding agents in this repo._

<!-- OMNI-MEMORY:START — auto-generated; edit outside this block only -->
## Project memory (OmniMemory)

_Auto-generated 2026-08-27 19:43 · 6 verified memories · default branch `main`._

This project has a persistent, branch-aware memory layer that stays fresh automatically (rebuilt at session start and after commits). **It is a reliable source of truth — pull it instead of guessing.** It is NOT force-fed into every prompt (that wastes tokens); fetch it when relevant.

- **Pull on demand:** before assuming any architecture — or when you need a decision, flow, gotcha, endpoint, or DB schema — run `omni-memory inject "<what you need>"` for the ranked **VERIFIED PROJECT MEMORY** block, and cite the `[id]`s you rely on. (`omni-memory recall "<q>"` is a lighter search.)
- The snapshot below orients you; treat it and pulled memory as verified truth. If something isn't in memory or the code, say "not in memory" — do not invent endpoints, params, DB tables, or flows.
- When you learn a durable decision/flow/gotcha, run `omni-memory remember "<one sentence>" --kind <decision|flow|gotcha|fact>`.
- Full knowledge base: `.omni-memory/MEMORY.md` · dashboard: `omni-memory ui`.

**Key decisions**
- Auth uses JWT in access_token cookie, refresh in httpOnly cookie  `[c40d73061ef8]`

**Gotchas**
- Additive kinds (gotcha/flow/…) are excluded so it doesn't cry wolf. 7 new tests + the full suite green on 3.9 and 3.11.  `[56399d6d0a2c]`
- never call payments.charge() before order row is committed  `[a6ce537e978b]`

**Flows**
- queue-operation: /resume  `[ecf4fdab432f]`
- POST /orders -> validate -> insert orders table -> publish order.created to Kafka -> 201  `[354c8701cd14]`

**API map**
- `omni_memory/serve.py` — the `/api/healthmap` endpoint  `[018767c2d466]` — `omni_memory/serve.py`

<!-- OMNI-MEMORY:END -->

---
> Source: [SinghAbhinav04/Omni-Memory](https://github.com/SinghAbhinav04/Omni-Memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
