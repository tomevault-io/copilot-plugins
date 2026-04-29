## agentic-workflow

> This file defines every agent role, the full orchestration loop, and how trigger phrases map to actions. Read this file at the start of every session.

# Agents

This file defines every agent role, the full orchestration loop, and how trigger phrases map to actions. Read this file at the start of every session.

---

## Trigger phrases

These phrases in Cursor chat start specific workflows:

| Phrase | Action |
|--------|--------|
| `import docs` | Run the `import-docs` skill. Read all PDF/Word/text files in `docs/`, extract content, populate `SCOPE.md`, and generate `memory/stack-guidance.md` from the confirmed architecture + tech stack. Ask user to confirm before saving. |
| `reqops` | Run the `reqops` skill to read SOW sources (prefer `docs/`) and produce/update per-feature requirement specs under `.pipeline/features/requirements/`. Does not write `TASKS.md`. |
| `figma ingest` | Run the `figma-plugin-ingest` skill. Use `plugin-figma-figma` to extract design/plugin artifacts and auto-enrich UI task fields in `TASKS.md`. |
| `parse scope` | Run the `parse-scope` skill. Read `SCOPE.md`, run ReqOps pre-pass when `.pipeline/sow.md` exists, then generate `TASKS.md`. Ask user to review before writing. Owns `TASKS.md`. |
| `delta scope` / `rescope` | Run the `delta-scope` skill. Diff the updated `SCOPE.md` against existing `TASKS.md`, produce a reviewable delta (`TASK-DELTA-XX` / `TASK-REMOVE-XX`), apply after user confirmation. Never silently re-plans. |
| `refresh mcp` | Force `fetch-mcp` / `figma-plugin-ingest` to ignore cache TTL and refetch. Use when upstream design/API changed. |
| `start build` | Run the `orchestrate` skill. Spawn dynamic parallel Builder agents by independent chains and keep a continuous Builder+QA loop running until all tasks are `done` (or blocked). |
| `resume build` | Re-read `TASKS.md`, restore in-flight states, and continue the same scheduler loop from where it stopped. |
| `qa only` | Run the `qa` skill on every task currently marked `in-review`. Do not start any new builds. |
| `show status` | Read `TASKS.md`. Print a summary table: done / in-review / in-progress / needs-fix / blocked / pending counts. List any blocked tasks with their blocker reason. |

---

## Reading order at session start

Every agent must read these files in this order before doing any work:

1. `SCOPE.md` — project definition, tech stack, feature breakdown
2. `TASKS.md` — current task list and statuses
3. `memory/architecture.md` — existing module map and data flow
4. `memory/patterns.md` — established code patterns to follow
5. `memory/decisions.md` — past decisions not to re-litigate
6. `memory/stack-guidance.md` — stack-specific implementation defaults for Builders

Skip files that do not exist yet (first run before scaffold).

---

## Agent roles

### Orchestrator

**Purpose:** Coordinate the build. Never writes feature code directly.

**Responsibilities:**
1. Read `TASKS.md` and identify all task groups.
2. Determine which task groups are independent (no shared files, no dependency edges between groups).
3. Assign each independent group to a Builder agent slot (Builder-A, Builder-B, Builder-C...).
4. Fan out Builders in parallel using the `orchestrate` skill.
5. Monitor `TASKS.md` continuously:
   - When a UI task has a Figma MCP URL and missing design artifact fields -> trigger `figma-plugin-ingest` before assigning to Builder.
   - When a UI task has `Design source: Figma` and missing design/codegen fields -> rerun `figma-plugin-ingest` (single unified path).
   - When a task moves to `in-review` → trigger QA agent on it.
   - When a UI task moves to `in-review` and has a design source -> QA runs standard + design fidelity checks in one pass.
   - When a task moves to `blocked` → surface blocker to the user immediately, do not continue past it.
 - When a task moves to `done` → check if any `pending` task is now dependency-ready and assign immediately.
 - When all tasks in a group are `done` → check if any previously blocked task is now unblocked and assign.
   - After every status transition, refresh Build progress counts in `TASKS.md` (helper: `python3 scripts/update-task-counts.py`).
6. When all tasks are `done` → report completion with a summary.

**Rules:**
- Never mark a task `done` — only the QA agent does that.
- Never skip a dependency. A task cannot start if any task in its `Depends on` list is not `done`.
- If two tasks touch the same file, they are not independent — serialize them or assign to the same Builder.

---

### Builder

**Purpose:** Implement one task group (a sequence of tasks for one feature).

**Responsibilities:**
1. Read the task block from `TASKS.md`.
2. Mark task `in-progress`.
3. If task has a MCP URL: run the `fetch-mcp` skill first. Read the cached output from `memory/mcp-cache/`.
4. Read `memory/patterns.md` — use established patterns; do not invent new ones unless the pattern doesn't exist.
5. Read `memory/architecture.md` — place new files in the correct module location.
6. Read `memory/stack-guidance.md` when it exists — use it for stack-specific architecture, UI, state, testing, and anti-pattern guidance.
7. Implement the vertical slice in this order: data model → API layer → shared types → UI → tests.
8. Run verification (see rules `01-verification.mdc`) before handing off.
9. Mark task `in-review`. Write a short implementation note in the task block under `QA notes:` for the QA agent.
10. Immediately pick up the next `pending` task in the same feature group (if any and if dependencies met).

**Rules:**
- Stay within the files listed in the task block. Do not modify files owned by another Builder's active task.
- Never mark a task `done`. Hand off to QA.
- If blocked (missing dependency, ambiguous spec, MCP URL unreachable): mark task `blocked`, write the reason, surface to Orchestrator.
- Follow all patterns in `memory/patterns.md` exactly. If a new pattern is established, add it after QA approval.
- Follow `memory/stack-guidance.md` when present for stack-specific defaults; do not drift into a different architecture style without an explicit project decision.
- If `Codegen artifact:` exists in the task block, implement from that scaffold first. Do not skip directly to custom UI coding unless the artifact is invalid.

---

### Design responsibilities (no standalone agent)

Design concerns are handled by the `figma-plugin-ingest` skill (primary path)
with `fetch-mcp` as a fallback. There is no standalone "Design Agent" role.

When a UI task has a Figma MCP URL:
1. Orchestrator (or the Builder on pickup) runs `figma-plugin-ingest` to
   extract design + plugin artifacts and auto-populate task design fields.
2. Builder implements from the resulting `Codegen artifact` and tokens.
3. QA verifies both standard gates and design-fidelity checks in a single
   pass (`04-design-fidelity.mdc`).

If `figma-plugin-ingest` cannot produce implementation-ready context, the
task is marked `blocked` with the exact missing node IDs or frames. No
separate design agent is needed.

---

### ReqOps Agent

**Purpose:** Convert SOW sources (prefer `docs/`) into traceable feature requirements. ReqOps writes only requirement files. `TASKS.md` is owned by `parse-scope`.

**Responsibilities:**
1. Read SOW sources from `docs/` first, then `SCOPE.md`, then `.pipeline/sow.md` fallback.
2. Produce/update per-feature requirements under `.pipeline/features/requirements/`.
3. Record contradictions against SOW as `GAP-XX`.
4. Provide acceptance-criteria-ready requirement inputs (with ReqOps IDs such as `[SD-01]`, `[AC-02]`) that `parse-scope` consumes when it generates `TASKS.md`.

**Rules:**
- If no authoritative SOW source exists across `docs/`, `SCOPE.md`, and `.pipeline/sow.md`, halt with explicit error.
- Never invent requirements not traceable to SOW.
- Define WHAT, not implementation HOW.
- Do not write `TASKS.md`. That is `parse-scope`'s responsibility.

---

### QA Agent

**Purpose:** Verify every completed task before it is marked done.

**Responsibilities:**
1. Pick up any task marked `in-review`.
2. Read the task's acceptance criteria checklist.
3. Run verification:
   - Lint on all files listed in the task block.
   - Typecheck on the whole project (not just changed files).
   - Run tests scoped to the feature being reviewed.
4. Review logic against acceptance criteria:
   - Each checklist item must be demonstrably satisfied.
   - Check for hardcoded values, missing error handling, unhandled edge cases.
5. If all checks pass:
   - Mark all checklist items `[x]`.
   - Mark task `done`.
   - Update `memory/architecture.md` if new modules or files were added.
   - Append new reusable patterns to `memory/patterns.md`.
   - Append any significant decisions to `memory/decisions.md`.
6. If any check fails:
   - Write specific, actionable fix notes in the task's `QA notes:` field.
   - Mark task `needs-fix`.
   - Do not touch `memory/` until the task passes.

**Rules:**
- Never approve a task with failing lint, typecheck, or tests — no exceptions.
- Never approve a task with unchecked acceptance criteria items.
- Fix notes must be specific: file path + line number + what is wrong + what is expected.

---

## Handoff protocol

```
Builder marks task in-progress
  → implements task
  → runs local verification (lint + types + tests)
  → writes QA notes (brief summary of what was built)
  → marks task in-review

QA Agent picks up in-review task
  → runs full verification
  → reviews acceptance criteria

  [pass]
    → marks all criteria [x]
    → marks task done
    → updates memory/

  [fail]
    → writes specific fix notes
    → marks task needs-fix

Builder picks up needs-fix task
  → reads QA notes
  → fixes issues
  → marks task in-review again (repeat until pass)
```

---

## Parallelism rules

- Tasks in different feature groups with no shared files and no dependency edge between them run in parallel.
- Tasks that share a file must be assigned to the same Builder and run sequentially.
- The scaffold task (TASK-000 or equivalent) must be `done` before any other task starts.
- Auth tasks must be `done` before any task that requires authenticated routes.
- Database schema tasks must be `done` before any task that reads/writes that schema.
- Builder pool is dynamic (`builder-1 ... builder-N`) based on independent ready chains; do not cap at 3.

---

## Memory update rules

After every `done` task, QA agent must:

1. **`memory/architecture.md`** — add any new modules, files, or data flows introduced by the task.
2. **`memory/patterns.md`** — append any new reusable pattern (component structure, API handler shape, test helper, etc.). Do not duplicate existing patterns.
3. **`memory/decisions.md`** — append any decision that was made during the task that future tasks should not re-litigate.

Keep all memory entries concise — bullet points and short code snippets only.

`memory/stack-guidance.md` is different: it is generated when `import docs` confirms the stack, and refreshed only when the declared architecture/tech stack changes materially.

---

## Error and blocker handling

- Follow `docs/BLOCKER-PLAYBOOK.md` for blocker note format, escalation message shape, and retry conditions.
- **Blocked task**: mark `blocked`, write the reason (e.g. "Depends on TASK-003 which is blocked"), surface to Orchestrator immediately.
- **MCP URL unreachable**: mark task `blocked`, note the URL that failed, ask user to verify or replace it.
- **Ambiguous spec**: mark task `blocked`, list exactly what is unclear, surface to user. Do not guess.
- **Persistent test failure**: after 2 failed fix attempts, mark `blocked`, summarize what was tried, ask user for direction.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
