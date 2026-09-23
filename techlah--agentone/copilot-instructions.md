## 99-task-completion-gate

> Mandatory task-done gate for ANY major task in this repo — structured AskQuestion before closing, then samectx and retrospective. Use after learn-knowledge storage, feature slices, research deliverables, refactors, or when workflow_run reaches task-completion-gate. Do not skip because the user said yes to research or store only.


# Task completion gate (mandatory for major tasks)

This rule **extends** `continuous-learning.mdc`. When its checklist applies, the gate is **required**, not optional agent discretion.

## Major task checklist

Treat work as a **major task** when **any** of these is true:

- The user invoked a **named workflow** (e.g. `learn-knowledge`) or `workflow_run` for a deliverable
- The user asked for a **coherent deliverable** (KB note, brief, multi-file change, integration) and substantive work is finished
- Acceptance criteria or stated scope for the session appear met

**Not** a major task: one-off Q&A, tiny single-line fix, or clearly mid-story work still in progress.

If unsure whether scope is complete, use the structured gate below — do not silently skip it.

## Two different confirmations

| User action | Allows |
| --- | --- |
| Research / “satisfied with summary” / “proceed to store” (including AskQuestion or chat for that step only) | `knowledge_write` or equivalent deliverable write **only** |
| **AskQuestion** option `mark-task-done` (task completion gate) | samectx sync + retrospective |

Never treat research/store **yes** or a generic chat **yes** as task completion. Plain *Can I mark this as done?* in chat is **discouraged** — it collides with other confirmations.

## Required sequence when a major task is complete

1. **Stop** and invoke **AskQuestion** with the options in `.cursor/workflows/prompts/_shared/task-completion-gate.md` (or equivalent labels: **Mark task done** / **Not yet** / **Pause here**).
2. Wait until the user selects **`mark-task-done`**. Do not run samectx or retrospective on bare **yes**, **done**, or **approve** unless they clearly chose **Mark task done** after this gate.
3. Run **samectx** skill → `samectx sync` (tasks, keypoints, decisions; no secrets).
4. Run **retrospective** skill → `./adr/`, `./knowledge/` as appropriate; present Retrospective Summary.

Applies to **ad-hoc major work** (no YAML workflow) and to workflow steps after **task-completion-gate**.

## Workflow runs

Prefer MCP **`workflow_run`** for named workflows so all steps (including completion gate) load into context. Follow steps **in order** through samectx and retrospective.

## Anti-patterns

- Ending the turn right after storage or “looks good” on research
- Treating any **yes** in chat as task-done after a different confirmation step
- Skipping retrospective because the change was “only docs”
- Deciding the rule “does not apply” without checking the checklist above

---
> Source: [TechLah/AgentOne](https://github.com/TechLah/AgentOne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
