## based-project-plan-methodology

> This file contains crucial workspace guidelines, build/test commands, and planning methodology instructions that **Claude Code** will automatically read and follow when launching in this repository.

# CLAUDE.md - Repository Guidelines for Claude Code

This file contains crucial workspace guidelines, build/test commands, and planning methodology instructions that **Claude Code** will automatically read and follow when launching in this repository.

---

## 🛠️ Build and Test Commands

When you need to compile code, run tests, or manage planning tasks, use these standard commands:

- **Build Project**: `pnpm run build`
- **Run Unit Tests**: `pnpm test`
- **Single Test File**: `node --import tsx --test <path_to_test>`
- **Global Planning CLI**: `plan <subcommand>` (or `pnpm run plan <subcommand>`)

---

## 🧭 Required Planning & Execution Workflow

You **must** execute all project activities by following the structured lifecycle defined in **[SKILL.md](file:///c:/Users/rlagu/workspace/antigravity-based-project-plan-methodology/SKILL.md)**. Use the planning CLI (`plan`) for all checklist state transitions.

### 1. Phase 1: Planning
- Before writing project code, run `plan init` to bootstrap `.agents/brain/project_thoughts/`.
- Interview the user to refine the implementation details.
- Edit `.agents/brain/project_thoughts/implementation_plan.md` to map out proposed changes.
- Validate your plan structure by executing:
  ```bash
  plan plan validate
  ```
- Obtain explicit user approval before proceeding to task breakdown.

### 2. Phase 2: Task Breakdown & Checklist
- Define implementation steps in `.agents/brain/project_thoughts/task_list.md`.
- Ensure all top-level tasks use explicit brackets and IDs: e.g. `- [ ] [task-1] Description`.

### 3. Phase 3: Execution and Boundaries (CRITICAL)
- **Status Updates**: Before starting a task, run `plan task status <taskId> in-progress`. Upon completion, run `plan task status <taskId> done`.
- **Enriching Subtasks**: If you need to add finer steps, use the safe subtask command:
  ```bash
  plan task add-subtask <parentTaskId> <subtaskId> "<subtaskText>"
  ```
- **Task Locks (Do Not Modify Confirmed Goals)**:
  - You are **strictly forbidden** from deleting, renaming, or modifying the description of any confirmed top-level task in `task_list.md` once execution has started.
  - Run `plan task validate` to ensure lock compliance.
  - If you believe a top-level task needs to be modified or redesigned, you **must halt work, explain the issue to the user, propose a better alternative, and obtain approval first.**

### 4. Phase 4 & 5: Verification & Walkthrough
- Once all tasks are marked complete, execute `plan test run` to run unit tests and log outcomes.
- Compile the final walkthrough report by running:
  ```bash
  plan walkthrough generate
  ```
- Point the user to the generated walkthrough at `.agents/brain/project_thoughts/walkthrough.md`.

---
> Source: [mostlytricks/based-project-plan-methodology](https://github.com/mostlytricks/based-project-plan-methodology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
