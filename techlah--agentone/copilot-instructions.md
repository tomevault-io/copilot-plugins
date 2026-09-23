## continuous-learning

> After major tasks, confirm done then samectx sync and retrospective asset consolidation


# Continuous learning (task completion gate)

Use this gate when a **major task** is complete. It complements DoD and feature acceptance; it applies to discovery, docs, and workshop work in this repo—not only shipped code.

## What counts as a major task

A coherent unit of work the user asked for that is **substantively finished**, for example:

- A user story or feature slice meeting its stated acceptance criteria
- A research or strategy deliverable (e.g. briefs, course outline, prompt pack)
- A multi-step refactor or integration that was the explicit scope of the session

Not a major task: a single clarifying answer, one-line fix, or mid-story step still in progress.

When the checklist in **`99-task-completion-gate.mdc`** applies, this gate is **mandatory** — do not skip because the model judges the task “minor” without that checklist.

## Research yes vs task-done yes

- **Research / store confirmation** (e.g. learn-knowledge step 2–3): authorizes writing the deliverable only.
- **Task-done confirmation** (this rule): authorizes samectx + retrospective only.

Never end a major task after research/store **yes** without the **task completion** gate (separate confirmation).

## How continuous-learning is triggered (read this)

Nothing runs samectx or retrospective **automatically**. Triggers are:

| Mechanism | When it fires |
| --- | --- |
| **Agent + AskQuestion** | Required: agent must call AskQuestion before samectx on every major task (this rule). |
| **`workflow_run` (learn-knowledge, etc.)** | YAML steps load **task-completion-gate → samectx → retrospective** into context; agent still executes them. |
| **Cursor hooks** (`.cursor/hooks.json`) | `stop` may send a follow-up if multiple file edits and gate not seen; `postToolUse` after `knowledge_write` injects next-step context. |
| **MCP `knowledge_write`** | Hook reminds: storage ≠ task done. |

Rules alone do **not** invoke tools. If the gate was skipped, the agent ended the turn without AskQuestion.

## Step 1 — Structured gate before closing

When you believe a major task is complete, **stop** and use **AskQuestion** (see `.cursor/workflows/prompts/_shared/task-completion-gate.md`):

- **Mark task done** — only this choice authorizes samectx + retrospective
- **Not yet** — continue the task
- **Pause here** — stop without sync or retrospective

Do **not** use a plain chat *Can I mark this as done?* or accept bare **yes** from an earlier research/store step. If they defer or redirect, continue the current work or the new scope they give.

## Step 2 — After confirmation only

Run in order:

### 2a. samectx sync

Follow the **samectx** skill:

1. Extract from the completed work: key tasks, keypoints, decisions (no secrets).
2. Run `samectx sync` with `--tasks`, `--keypoints`, and `--decisions` (semicolon-separated lists).
3. Notes land under `samectx-notes/` in this project; keep content factual and non-sensitive.

### 2b. Retrospective — consolidate process assets

Follow the **retrospective** skill:

1. Review what went well, what was hard, and reusable lessons.
2. Persist durable assets:
   - **ADRs** → `./adr/` when a non-obvious decision was made
   - **Knowledge** → `./knowledge/` (e.g. `ops/`, `methods/`, `insights/`) for reusable research, ops, or domain notes
3. Update `knowledge/README.md` index when adding notable knowledge docs.
4. Present a **Retrospective Summary** (ADRs, knowledge, skipped items). An empty retrospective is valid if nothing is worth persisting.

Do not commit unless the user requests a commit.

## Anti-patterns

- Assuming completion without asking
- Running samectx or retrospective before user confirmation
- Skipping retrospective because the task was “only docs” when lessons or decisions are worth keeping

---
> Source: [TechLah/AgentOne](https://github.com/TechLah/AgentOne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
