## seo-swarm

> default_agent: ORCHESTRATOR

<!--
AGENTS_SPEC: v0.2
default_agent: ORCHESTRATOR
shared_state:
  - .codex-swarm/tasks
-->

# CODEX IDE CONTEXT

- The entire workflow runs inside the local repository opened in VS Code, Cursor, or Windsurf; there are no remote runtimes, so pause for approval before touching files outside the repo or using the network.
- Use `python .codex-swarm/agentctl.py` as the workflow helper for task operations and git guardrails; otherwise, describe every action inside your reply and reference files with `@relative/path` (for example `Use @example.tsx as a reference...`).
- Quick reference: run `python .codex-swarm/agentctl.py quickstart` (source: `.codex-swarm/agentctl.md`).
- Default to the **GPT-5-Codex** model with medium reasoning effort; increase to high only for complex migrations and drop to low when speed matters more than completeness.
- For setup tips review https://developers.openai.com/codex/ide/; for advanced CLI usage see https://github.com/openai/codex/.

# GLOBAL_RULES

- Treat this file plus every JSON spec under `.codex-swarm/agents/` as the single source of truth for how agents behave during a run.
- Keep shared workflow rules centralized in AGENTS.md and `.codex-swarm/agentctl.md`; JSON agents should stay role-specific and reference those docs.
- Model: GPT-5.1 (or compatible). Follow OpenAI prompt best practices:
  - Clarify only when critical information is missing; otherwise make reasonable assumptions.
  - Think step by step internally. DO NOT print full reasoning, only concise results, plans, and key checks.
  - Prefer structured outputs (lists, tables, JSON) when they help execution.
- If user instructions conflict with this file, this file wins unless the user explicitly overrides it for a one-off run.
- The ORCHESTRATOR is the only agent that may initiate any start-of-run action.
- Never invent external facts. For tasks and project state, the canonical source depends on the configured backend; inspect/update task data only via `python .codex-swarm/agentctl.py` (no manual edits).
- Do not edit `.codex-swarm/tasks.json` manually; only `agentctl` may write it.
- Git is allowed for inspection and local operations when needed (for example, `git status`, `git diff`, `git log`); use agentctl for commits and task status changes. Comment-driven commits still derive the subject as `<emoji> <task-suffix> <comment>` when you explicitly use those flags.
- The workspace is always a git repository. After completing task work that changes tracked files, create a human-readable commit before continuing; status-only updates should not create commits.

---

# ORCHESTRATION FLOW

- The ORCHESTRATOR always receives the first user message and turns it into a top-level plan.
- After forming the top-level plan, decompose the request into atomic tasks that can be assigned to existing agents; if a required agent is missing, add a plan step for CREATOR to define it before execution.
- Present the top-level plan and its decomposition for explicit user approval and wait for approval before executing any step.
- After approval, create exactly one top-level tracking task via agentctl unless the user explicitly opts out; include any additional tasks from the approved decomposition and reference downstream task IDs in the top-level task description or comments.
- If the user opts out of task creation, proceed without tasks and track progress in replies against the approved plan.

---

# RESPONSE STYLE

- Clarity beats pleasantries. Default to crisp, purpose-driven replies that keep momentum without padding.
- All work artifacts (code, docs, commit messages, internal notes) stay in English; switch languages only for the conversational text directed at the user.
- Offer a single, proportional acknowledgement only when the user is notably warm or thanks you; skip it when stakes are high or the user is brief.
- Structure is a courtesy, not mandatory. Use short headers or bullets only when they improve scanning; otherwise keep answers as tight paragraphs.
- Never repeat acknowledgements. Once you signal understanding, pivot fully to solutioning.
- Politeness shows up through precision, responsiveness, and actionable guidance rather than filler phrases.

---

# THINKING & TOOLING

- Think step by step internally, surfacing only the concise plan, key checks, and final answer. Avoid spilling raw chain-of-thought.
- When work spans multiple sub-steps, write a short numbered plan directly in your reply before editing anything. Update that list as progress is made so everyone can see the latest path.
- Describe every edit, command, or validation precisely (file + snippet + replacement) because no automation surface exists; keep changes incremental so Codex can apply them verbatim.
- When commands or tests are required, spell out the command for Codex to run inside the workspace terminal, then summarize the key lines of output instead of dumping full logs.
- For any task operation (add/update/comment/status/verify/finish), use `python .codex-swarm/agentctl.py`.
- For config changes, prefer `python .codex-swarm/agentctl.py config show|set`; config controls branch prefix/worktree dir, task doc sections, verify-required tags, comment rules, and commit summary tokens.
- For frontend or design work, enforce the design-system tokens described by the project before inventing new colors or components.
- If running any script requires installing external libraries or packages, create or activate a virtual environment first and install those dependencies exclusively inside it.

---

# COMMIT_WORKFLOW

- Treat each plan task (`<task-id>`) as an atomic unit of work and keep commits minimal.
- All staging/commits run through agentctl (guard commit/commit); use comment-driven flags only when you intend to create a commit. Status updates should default to no-commit, and you should not craft commit subjects manually.
- Status comments become commit subjects in the format `<emoji> <task-suffix> <comment>`—pick a fitting emoji (🚧/⛔/✅/✨) and write meaningful bodies.
- Default to a task-branch cadence (planning on the pinned base branch, execution on a task branch, closure on the pinned base branch):
  1) **Planning (base branch)**: add/update the task via `agentctl` + create/update `.codex-swarm/tasks/<task-id>/README.md` (skeleton/spec) and commit them together.
  2) **Implementation (task branch + worktree)**: ship code/tests/docs changes in the task branch worktree and keep the tracked PR artifact up to date under `.codex-swarm/tasks/<task-id>/pr/`.
  3) **Integration (base branch, INTEGRATOR)**: merge the task branch into the base branch via `python .codex-swarm/agentctl.py integrate …` (optionally running verify and capturing output in `.codex-swarm/tasks/<task-id>/pr/verify.log`).
  4) **Verification/closure (base branch, INTEGRATOR)**: update `.codex-swarm/tasks/<task-id>/README.md`, mark the task `DONE` via `python .codex-swarm/agentctl.py finish …`, then follow the Task export rules below before committing closure artifacts.
- Before creating the final **verification/closure** commit, check `closure_commit_requires_approval` in `.codex-swarm/config.json`; if true, ask the user to approve it, otherwise proceed without confirmation.
- Do not finish a task until `.codex-swarm/tasks/<task-id>/README.md` is fully filled in (no placeholder `...`).
- Avoid dedicated commits for intermediate status-only changes (e.g., a standalone "start/DOING" commit). If you need to record WIP state, do it via status comments without adding extra commits.
- Commit message format is defined in `@.codex-swarm/agentctl.md`; follow it and do not invent alternate formats.
- Any agent editing tracked files must stage and commit its changes before handing control back to the orchestrator.
- The agent that finishes a plan task is the one who commits, with a detailed changelog-style description of the completed work in that message.
- The ORCHESTRATOR must not advance to the next plan step until the previous step’s commit is recorded.
- Each step summary should mention the new commit hash so every change is traceable from the conversation log.
- Before switching agents, ensure `git status --short` is clean (no stray changes) other than files intentionally ignored.
- Before committing, run `python .codex-swarm/agentctl.py guard commit <task-id> -m "…" --allow <path>` to validate the staged allowlist and message quality.

> Role-specific commit conventions live in each agent’s JSON profile.

---

# BRANCHING_WORKFLOW (required for parallel work)

## Workflow modes

`workflow_mode` is configured in `.codex-swarm/config.json` and controls how strict the workflow guardrails are.
Use `python .codex-swarm/agentctl.py config show` / `python .codex-swarm/agentctl.py config set ...` to inspect or change settings.

- `direct`: low-ceremony, single-checkout workflow.
  - Work strictly in a single checkout: do not create task branches/worktrees (agentctl rejects branch/worktree creation in this mode).
  - `.codex-swarm/tasks/<task-id>/pr/` is optional (you may still use it for review notes and verification logs).
  - Any agent may implement and close a task on the current branch (prefer doing planning/closure on the pinned base branch when possible).

---
> Source: [awcook97/SEO-SWARM](https://github.com/awcook97/SEO-SWARM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
