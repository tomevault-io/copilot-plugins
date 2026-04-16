## test

> - Be concise; use short bullet lists; avoid long explanations and back-and-forth.


- Communication
  - Be concise; use short bullet lists; avoid long explanations and back-and-forth.
  - Ask the user questions only when strictly necessary to proceed safely or when a decision has significant impact.
  - Prefer to infer reasonable defaults from the repository (docs, code, orga files) and document assumptions in the task's `plan.md` or `notes.md` instead of asking.
  - Keep detailed plans and decisions in `orga/tasks/.../plan.md` (or `notes.md`), and use chat mainly to report status, results, and next steps.

- Autonomy
  - Work as independently and continuously as possible within the current task, without waiting for user confirmation between small, safe steps.
  - Batch related tool calls and edits where possible, then summarize the outcome briefly.

- Workflow
  - On a new task, first read `orga/tasks/TASKS.md` and the task's `README.md`/`plan.md` before asking anything.
  - Only interrupt the user when blocked by missing information, when there is a real safety concern, or when a strategic choice is required.
  - When a user invokes a workflow, you must:
    - Create and maintain an explicit per-step checklist (use `todo_list`).
    - Record progress in a durable repo artifact (task `plan.md`/`notes.md` when task-related; otherwise `orga/domain/init-run.md` or another clearly named `orga/domain/*` file).
    - Not declare the workflow complete unless every step is done or explicitly skipped with justification.
    - Close with an evidence-based completion report (workflow step -> evidence such as file paths, commands run, and `git status` outcome).
  - If you change any runtime Windsurf files under `.windsurf/` (rules, workflows, skills), you must:
    - Update the corresponding reference files under `docs/windsurf/`.
    - Explicitly tell the user to copy/sync `docs/windsurf/` into `.windsurf/`.
  - When a task touches **UX**, **permissions/roles**, or other **security-sensitive** areas, ensure the plan and implementation include:
    - A clear list of affected screens/flows (for UX) and a brief design/UX review.
    - An explicit mapping of who should be able to see/do what (for permissions/roles) plus tests and manual "attack" attempts.
    - A short threat sketch and targeted tests or smoke checks for security-sensitive changes.
  - Treat any end-of-message status blocks as concise pointers to work recorded in the repo (e.g. which sections of `plan.md` / `outcome.md` were updated), not as the primary place to store detailed reasoning.

- Commands & tests
  - Form commands as **bare shell commands** without prompts or decorations.
  - Never use `cd` inside commands. Instead, set the working directory via the tool (Cwd) to the project root when running project commands.
  - Prefer `python` as the Python executable:
    - Example – full test suite: `python -m pytest -q`.
    - Example – subset: `python -m pytest -q tests/test_mvp_flows.py::test_admin_applications_list_decision_filter_and_indicators`.
  - Assume the human maintainer has activated their virtual environment (if any) in the IDE terminal so that `python` points at the correct interpreter.
  - If a `python ...` command fails with an obvious launcher error (e.g. `python: command not found` on Windows), prefer `py -m pytest ...` in future commands for this repo and record that assumption in the relevant task's `plan.md`.
  - Use quiet flags like `-q` for pytest when suitable to keep output manageable, especially for targeted test runs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmilyMacro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
