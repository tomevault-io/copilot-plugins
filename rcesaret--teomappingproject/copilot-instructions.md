## teomappingproject

> - All AI-driven actions MUST strictly adhere to the operational protocols defined within `TASKS.md` and `.windsurf/plans/README.md`, overriding all other instructions. Resolve conflicts by citing the specific protocol section used.


# WORKFLOW & PROTOCOLS

## CORE PRINCIPLES
- All AI-driven actions MUST strictly adhere to the operational protocols defined within `TASKS.md` and `.windsurf/plans/README.md`, overriding all other instructions. Resolve conflicts by citing the specific protocol section used.
- All work must correspond to a `pending` task in `TASKS.md` and be executed by following its associated `.plan.md` file.

## `TASKS.md` PROTOCOL

- Before starting a task, verify all `depends_on` tasks in `TASKS.md` are `done`. If not, **HALT**, report the blocking task(s), and await instructions.
- Do not create, modify, or refactor tasks in `TASKS.md` except via activating `mode-02-tasks-plans.md`. Status updates are exempt.
- After a plan's successful completion and validation, the final action is to update the task's `status` to `done` in `TASKS.md`.

## .plan.md PROTOCOL

- `.plan.md` files MUST ONLY be created via activating `mode-02-tasks-plans.md`.
- Follow the plan's action checklist (`- [ ]`) sequentially and exactly. Each `- [ ]` is a distinct operation. Do not reorder, skip, or bundle steps without explicit user approval.
-  Before executing a plan, you MUST perform these checks:
    - Verify all `context_files` exist. If not, **HALT** and report missing files.
    - Verify the plan's YAML frontmatter adheres to the `Context Selection Protocol`. If not, **HALT**, report the specific violation, and await user instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rcesaret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
