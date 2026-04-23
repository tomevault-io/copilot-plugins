## plann

> Whenever you use this rule, start your message with the following:

# AI Task Management System Overview

Whenever you use this rule, start your message with the following:

"Accessing AI Task Management system overview..."

The AI Task Management system is a file-based project management and AI agent operational framework designed to plan features, manage development tasks, and maintain a memory of past work. It consists of three main components, each governed by its own detailed rule file:

1. **Plans (`@planning.mdc`)**:
    * **Purpose**: Defines how Product Requirements Documents (PRDs) are created and structured for the overall project and specific features.
    * **Location**: PRDs are stored in `.cursor/plans/`, with feature-specific plans in `.cursor/plans/features/`.
    * **Key File**: A global `PLAN.md` is mandatory in `.cursor/plans/`.
    * **Details**: For plan creation, fully review [planning.mdc](mdc:.cursor/rules/planning.mdc)

2. **Tasks (`@task-management.mdc`)**:
    * **Purpose**: Governs the creation, management, and lifecycle of individual development tasks.
    * **Active Tasks**: All active tasks reside in `.cursor/plans/` as `task{id}_name.md` files.
    * **Master View**: A master checklist, `.cursor/TASKS.md`, mirrors the status of tasks in the `.cursor/plans/` directory and must be kept synchronized.
    * **Details**: For task creation, fully review [task-management.mdc](mdc:.cursor/rules/task-management.mdc)

3. **Memory (`@active-memory.mdc` & `@archive-memory.mdc`)**:
    * **Purpose**: Maintains current session context (active memory) and archives completed/failed tasks to provide historical context (archive memory).
    * **Location**: Active memory files in `.cursor/memory/` (active-context.md, requirements.md, architecture.md, work-plan.md). Archived task files stored in `.cursor/memory/tasks/`.
    * **Log Files**: Task archive log in `.cursor/memory/task-log.md`. Plan archive log in `.cursor/memory/plan-log.md`.
    * **Details**: For memory management, fully review [active-memory.mdc](mdc:.cursor/rules/active-memory.mdc) and [archive-memory.mdc](mdc:.cursor/rules/archive-memory.mdc)

This interconnected system allows for structured project development, from high-level planning to task execution and historical review, primarily managed through Markdown files and defined agent responsibilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sangdth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
