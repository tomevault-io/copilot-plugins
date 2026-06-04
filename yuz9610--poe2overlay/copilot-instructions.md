## workflow

> Mandatory AI workflow — canonical rules in PROJECT_WORKFLOW.md; five hard rules only here


# Workflow enforcement

**Canonical (edit here first):** [`docs/PROJECT_WORKFLOW.md`](docs/PROJECT_WORKFLOW.md) § Agent 強制規則.  
**Prompts:** [`docs/AI_AGENT_PROMPTS.md`](docs/AI_AGENT_PROMPTS.md). **Human cheat sheet:** [`docs/AI_AGENT_SETUP.md`](docs/AI_AGENT_SETUP.md).
**Docs ownership:** [`docs/README.md`](docs/README.md#目標信息架構). Workflow rules belong here in `PROJECT_WORKFLOW.md`; this rule is summary only.

When the user uses a **workflow short command**, follow the matching step in `AI_AGENT_PROMPTS.md`. Do not improvise a shorter path.

## Five hard rules (summary only — do not expand here)

1. **First response = proposal only** (read-only exploration OK). No writes, no commit, no `Done`.
2. **Skip gate** only if the user says in the **same message**: `跳过确认` / `跳過確認` / `skip confirmation` / `直接修改` / `直接执行` / `直接實作` / `不用等确认` / `不必等確認`. Vague urgency or `In progress` / `Planned` does **not** skip.
3. **`執行 task`** → Phase A execution plan first; Phase B after human confirms. **`In progress` does not skip Phase A.**
4. **Step 4 never commits.** **`驗收 task`** → acceptance review first; agent **reads `logs/*.log`**; commit only after human confirms **Accept** (plus log downgrade).
5. **Changing workflow rules** → update `PROJECT_WORKFLOW.md` first, then sync `AI_AGENT_PROMPTS.md` intro, `AGENTS.md`, `AI_AGENT_SETUP.md`, this file.

## Step 4 / Step 5 pointers

| Step | First reply | After confirm |
|------|-------------|---------------|
| 4 Execute | Plan only | Implement; fill task doc; **no commit** |
| 5 Accept | `Accept` / `Needs changes` / `Blocked` | Done + commit per [§ Commit Message](docs/PROJECT_WORKFLOW.md#commit-message); `Co-authored-by: CursorAgent`; Windows `git commit -F` UTF-8 |

Step 3: new sprint → `SPRINTS.md` **Current Sprint** only (`Planned`). Step 6: `SUMMARY.md` per [sprints/README.md § SUMMARY template](docs/sprints/README.md#summarymd-模板-step-6); handoff IDs must exist in `SPRINTS.md` Task List; include workflow / project / docs review; update `SPRINTS.md` Current Snapshot, parent item status, Task List status / sprint fields, and reorder Above The Line Items / Task List with the same item order (in progress / started, then not started, then done); send larger improvements back to brainstorm / review / planning.

---
> Source: [yuz9610/POE2Overlay](https://github.com/yuz9610/POE2Overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
