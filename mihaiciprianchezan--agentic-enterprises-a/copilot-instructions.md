## agentic-enterprises-a

> Context for Claude Code (or any agent) picking up this repo. Read this first.

# CLAUDE.md — Agent-Native Cell

Context for Claude Code (or any agent) picking up this repo. Read this first.

## What this is

One sovereign **cell** of the Agent-Native Enterprise model — the smallest complete unit
of the architecture, scoped around a **software-delivery workflow**: intake a feature/bug
request → produce a code change → verify it → hand the change back to the existing human
review/merge process. It is **not** a federation; one cell first, by design.

The full design lives in `docs/` and is the source of truth. Code must conform to it.

## Read the design before coding (in this order)

0. `docs/Anatomy-of-a-Run.md` — the operational overview: one ticket → verified PR, role by role. Start here for the big picture.
1. The model — https://github.com/MihaiCiprianChezan/Agentic-First-Enterprises (the canonical
   spec repo). The 11 design invariants in §1 are non-negotiable.
2. `docs/One-Cell-Build-Plan.md` — the build plan and milestone order (M0–M9).
3. `docs/Cell-Constitution.md` — the constitution this cell runs under (Articles 1–11).
4. `docs/Role-Contracts.md` — the seven role contracts (four operating + three system roles).
5. `docs/Handbrake-Interface.md` — the control plane (M4).
6. `docs/Build-Spec.md` — **the spec code is built against**: schemas (§1–2), trace/cost (§3),
   the idempotency wrapper (§4), and the governance rule set R1–R12 (§5).
7. `docs/Component-Selection.md` — capability → tool mapping; what's built vs. deferred.

## The invariants that constrain every line of code

- **#1** Depend on the contract, not the implementer. Bind to the Protocols in `cell/roles/contracts.py`.
- **#3** Every flow has a handbrake (pause/inspect/inject/resume/replay). Structural, not a feature.
- **#4** Side effects are safe to retry: exactly-once for reversible, **at-most-once** for irreversible.
  The outside world is never assumed idempotent. All effects go through `cell/effects/wrapper.py`.
- **#5** State lives outside the actor. Nothing durable in an agent's memory — it goes to the event plane.
- **#9** A human who takes over a Role is bound by that Role's authority, not their human Office.
- **#10** Governance is compiled from the constitution; agents never author their own rules.

## Build status: M0–M9 complete

The whole one-cell build plan is **implemented, reviewed, and merged** (the full suite is green).
All milestones are done:

- **M0** durable hash-chained event store + idempotent-action wrapper (exactly-once / at-most-once
  across a kill-and-resume — the acceptance tests in `tests/test_m0_idempotency.py` and the
  whole-cell gate in `tests/test_full_flow_kill_resume.py` pass).
- **M1** ratified constitution · **M2** role contracts · **M3** observability (trace + cost) ·
  **M4** the handbrake · **M5** governance compiled R1–R12 · **M6** autonomy graduation ·
  **M7** the Steward · **M8** the Optimizer (capability/cost routing) · **M9** the Auditor
  (Article 11 amendment + rate/report + the suspend-and-escalate breaker).
- Plus: the composition harness (`Cell.assemble`), the real CLI-agent runtime (`claude -p` in the
  Executor seat), the observability inspector (`python -m cell.observe`), cost-into-events (measured
  wall-clock + reported tokens), `start()` re-entry, and the version layer (`cell/versions.py`).

**Only optional federation / the supra-constitution remains** — and its precondition is a *second
cell*. Per invariant #8, it is not built until one exists.

> Picking up work? Read `README.md` for the current onramp and `docs/Using-a-Cell.md` for usage.
> `docs/M0-Implementation-Notes.md` is retained as the historical M0 playbook.

## Project layout

```
src/cell/
  domain/objects.py     # wire schema: Ticket, Goal, WorkItem, Output, Verdict (Build-Spec §1)
  roles/contracts.py    # the five role Protocols (M2)
  roles/reference.py    # reference implementers of the operating roles (M2)
  flow.py               # run_flow — composes the role contracts + traces them (M2/M3)
  handbrake.py          # CellHandbrake — the five control primitives on the flow (M4)
  runtime/              # bind a real CLI coding agent to the Executor seat (sub-project B)
  live.py               # opt-in live real-slice runner (CELL_LIVE=1 python -m cell.live)
  cell.py               # Cell — composition root; wires every plane + the live governance gate
  demo.py               # runnable end-to-end demo of the §7 definition of done (python -m cell.demo)
  autonomy.py           # AutonomyBoard — Board-ratified ceiling amendments (M6)
  steward.py            # Steward — drift/loop/cost quarantine + rollback (M7)
  optimize.py           # Optimizer — capability/cost-aware implementer routing (M8)
  versions.py           # version registry (event-sourced) + per-version scorecard — the M9 precondition
  auditor.py            # Auditor — rate versions + report (M9b) + suspend-and-escalate breaker (M9c)
  observe.py            # read-only run inspector over the durable event plane (python -m cell.observe)
  planes/
    memory.py           # event/memory plane — EventStore + Event/Checkpoint/Decision (M0)
    observability.py    # observability plane — TraceSpan + cost attribution (M3)
    governance.py       # action-class registry + RuleSetGovernance R1–R12 (M5)
    control.py          # the Handbrake Protocol — interface for M4 (impl in handbrake.py)
  effects/wrapper.py    # the idempotency wrapper — M0 CORE
tests/                  # one or more suites per milestone (M0–M9) + cross-cutting suites
```

## Conventions

- **The docs are authoritative.** If code and a doc disagree, the doc wins — or amend the doc
  first (it is a living constitution; changes are tracked, per Build-Spec §5.4).
- **The assembled cell gates on the compiled rules.** `Cell.assemble()` wires `RuleSetGovernance`
  as the live R6 gate at the action site; `PermissiveGovernance` is a dev-only stub. The
  composition root (`cell/cell.py`) is the single seam where a real role-runtime binds.
- **Trace every claim to a clause.** Governance rules and gates cite a constitution Article;
  keep that traceability when you implement them (Build-Spec §5.4 validation/attestation).
- **YAGNI / invariant #8.** The Optimizer (M8) and Auditor (M9) are built now that their
  preconditions exist. Do **not** build a message broker or a second cell / federation — deferred
  until their precondition (a second cell) exists (Constitution Art. 3.4).
- **Tool-agnostic until it isn't.** Pick concrete tools per `Component-Selection.md`; keep them
  behind the interfaces so they stay swappable (invariant #1).
- **Single-threaded per flow.** One writer per flow at a time (a resumed flow is a new writer
  after the old one died, never a concurrent one). The durable store's UNIQUE(flow_id, seq)
  key is the backstop: a racing second writer fails loudly instead of forking the chain.
  See the module docstring in `cell/planes/memory.py`.

## How to run

```bash
pip install -e ".[dev]"
pytest            # the full suite is green (M0–M9 complete)
ruff check src tests && ruff format --check src tests && mypy   # the CI quality gates
```

CI enforces all three on every PR (`.github/workflows/test.yml`); keep them green before
pushing. mypy is strict on `src/cell` (bare `dict`/`list` generics allowed — wire payloads
are open dicts by design).

## Build order (executed in full)

M1 (Board + constitution) → M2 (role contracts) → M3 (observability) → M4 (handbrake) →
M5 (governance R1–R12) → M6 (autonomy graduation) → M7 (Steward) → M8 (Optimizer) → M9 (Auditor).
See `docs/One-Cell-Build-Plan.md` for the authoritative milestone definitions.

> Note: adopting a durable-execution engine (checkpointer/replay) is a **tooling choice**
> that implements the M0 checkpointer and the M4 handbrake replay — not a milestone of its
> own. The M0 foundations already externalize state into the durable event store + ledger.

---
> Source: [MihaiCiprianChezan/Agentic-Enterprises-A](https://github.com/MihaiCiprianChezan/Agentic-Enterprises-A) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
