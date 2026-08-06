## latam-last-mile-logistics-work-sample

> > This file serves as the **entry point** for any agent operating within this

# AGENTS.md — Navigational Map for AI Agents

> This file serves as the **entry point** for any agent operating within this
> repository. It is NOT a comprehensive rulebook: it is a **map**. Read only what you
> require, precisely when you require it (progressive disclosure).

---

## 1. Before Starting (Mandatory)

1. Execute `./init.sh` and verify that it terminates without errors. If it fails, **stop**
   and resolve the environment before altering any code.
2. Read `progress/current.md` to comprehend the state left by the preceding session.
3. Read `feature_list.json`. Any new feature (`"sdd": true`) must strictly undergo
   **Spec Driven Development** — refer to `docs/specs.md` and §4 of this document.
4. Read `docs/specs.md` prior to interacting with any spec or feature marked `sdd: true`.

## 2. Repository Map

| File / Directory             | Contents                                                                    | When to read it |
|------------------------------|-----------------------------------------------------------------------------|---------------|
| `feature_list.json`          | Task ledger detailing state (`pending` / `spec_ready` / `in_progress` / `done` / `blocked`) | Always, at inception |
| `progress/current.md`        | State of the active session                                                 | Always, at inception |
| `progress/history.md`        | Append-only chronological log of prior sessions                             | When historical context is requisite |
| `specs/<feature>/`           | `requirements.md` + `design.md` + `tasks.md` (Kiro-style)                   | Prior to implementing any feature with `"sdd": true` |
| `docs/architecture.md`       | Defines what constitutes "quality work" within this project                 | Prior to implementation |
| `docs/conventions.md`        | Rules governing styling, nomenclature, and structural paradigms             | Prior to authoring code |
| `docs/specs.md`              | SDD Protocol: EARS notation, the 3 core files, human approval gate          | Prior to drafting or interpreting a spec |
| `docs/verification.md`       | Methodology to prove functional correctness (includes requirement traceability)| Prior to declaring a task as `done` |
| `CHECKPOINTS.md`             | Objective criteria denoting a "correct final state"                         | For rigorous self-evaluation |
| `.antigravitycli/agents/`            | Subagent definitions (`leader`, `spec_author`, `implementer`, `reviewer`)   | When orchestrating workflows |
| `src/`                       | Application source code                                                     | For implementation |
| `tests/`                     | Automated test suites                                                       | For verification |

## 3. Hard Rules (Non-Negotiable)

- **Strictly one feature at a time.** Never conflate modifications from multiple tasks within the same session.
- **Never declare a task `done` without green tests.** Execute `./init.sh` and
  ensure the test suite passes at 100%.
- **Do not bypass the spec phase.** Every feature with `"sdd": true` must
  be processed by `spec_author` and secure human approval prior to touching code.
- **Do not bypass the human approval gate.** The leader halts the workflow
  at `spec_ready` and awaits explicit authorization.
- **Document your actions** in `progress/current.md` synchronously as you work, not asynchronously at the end.
- **Leave the repository pristine** prior to session closure (refer to §5).
- **If you lack knowledge, consult `docs/`** rather than fabricating a solution.

## 4. Workflow (SDD)

```
pending → [spec_author] → spec_ready → ⏸ HUMAN → in_progress → [implementer → reviewer] → done
```

1. The leader detects the first `pending` feature flagged with `"sdd": true`.
2. The leader dispatches `spec_author`, which authors
   `specs/<name>/{requirements,design,tasks}.md` and advances the status to
   `spec_ready`.
3. **Pause.** The human reviews the spec in `specs/<name>/` and approves (or mandates revisions).
4. Once approved, the leader transitions the status to `in_progress` and dispatches `implementer`.
5. The implementer executes `tasks.md` sequentially, marking them `[x]`.
6. The reviewer validates traceability `R<n>` ↔ test and confirms task completion;
   it subsequently approves or rejects the implementation.
7. Upon approval, the implementer marks the feature `done` and appends the summary to
   `progress/history.md`.

## 5. Session Closure (Lifecycle)

Prior to terminating:

1. Execute `./init.sh` — verify a completely green state.
2. If the task is finalized: mark `status: "done"` in `feature_list.json`.
3. Append the summary from `progress/current.md` to the bottom of `progress/history.md`.
4. Purge `progress/current.md`, retaining only the bare template.
5. Do not leave behind temporary files, stray `print()` debug statements, or contextless TODOs.

## 6. Blockage Resolution

- Re-read the pertinent section within `docs/`.
- If a tool fails to yield the anticipated outcome, **do not fabricate a workaround**:
  document the blockage rigorously in `progress/current.md` and halt the session.

---
> Source: [Many871027/LATAM-Last-Mile-Logistics---WORK-SAMPLE](https://github.com/Many871027/LATAM-Last-Mile-Logistics---WORK-SAMPLE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
