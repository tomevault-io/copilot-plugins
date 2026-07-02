## notes

> How do agent conversations differ from Ask or Edit?

How do agent conversations differ from Ask or Edit?

Overview

- Agent: multi-step tasks with autonomy, plans, tools, diffs/commits.
- Ask: Q&A and explanations; no file mutations or commands.
- Edit: precise code transforms; small, reviewable diffs.

Side-by-Side Comparison

- Scope: Agent → plan → execute → verify across files/modules. Ask →
  explain/ideate. Edit → scoped code change.
- Autonomy: Agent can read/write many files, run commands, call tools, open PRs.
  Ask does not act. Edit mutates code you specify.
- Planning/State: Agent tracks TODOs and iterates until done. Ask/Edit are
  single-turn, stateless.
- Execution: Agent can run sync or in background with progress updates. Ask/Edit
  complete immediately.
- Output: Agent produces tasks, logs, and diffs/commits. Ask returns
  explanations/snippets. Edit yields a concrete patch.
- Reviewability: Edit is small/local and easy to diff. Agent may span
  files/projects and needs PR review. Ask has nothing to review.
- Safety/Permissions: Agent requests approval for risky actions (commands,
  network, large rewrites). Ask/Edit operate inside the editor without external
  side effects.
- Latency/Cost: Agent slower but reduces coordination on big tasks. Ask/Edit are
  fast for answers and small refactors.
- Best For: Agent → refactors/features/migrations. Ask → clarification/ideas.
  Edit → deterministic, scoped edits.

When To Use Each

- Agent: Feature/refactor/migration across modules; needs plan, multiple edits,
  tests/PR; wants progress and checkpoints.
- Ask: Explanation, API usage, debugging hypotheses, small snippets; no file
  changes.
- Edit: Exact change known (rename, rewrite a function, add types), especially
  within a selection/single file.

Prompting Templates

- Agent: Goal, constraints, acceptance criteria, boundaries (files to
  touch/avoid), timebox, test scope. Ask for a short plan, then approve.
- Ask: Provide file/symbol context, stack traces, versions, desired depth (quick
  vs deep dive).
- Edit: Select code; describe the change and intent (e.g., convert callback to
  async/await; no behavior change).

Examples

- Agent: Migrate from request → undici, update call sites, add wrapper, open PR
  with changelog.
- Ask: Why does fetch return 422 here? What should I log?
- Edit: In this file, replace EventEmitter with EventTarget; keep tests green.

Operational Notes (VS Code)

- Agents may create/update multiple files, run scripts, and start background
  tasks. Expect progress notes and final summary/PR.
- Edits appear as inline diffs to accept/reject quickly.
- Ask responses never modify files; paste results manually if desired.
- Prefer scoping agents (folders/files/commands). Ask for their plan first;
  approve stepwise for risky actions.

Safety & Permissions

- Approvals: Agents should request approval before running shell commands,
  accessing the network, or opening PRs.
- Boundaries: Specify folders/files to touch or avoid; cap runtime with a
  timebox.
- Reviews: Prefer small, reviewable diffs; use PRs for multi-file changes.
- Observability: Expect progress updates (plan, steps, logs) and a final summary
  of changes.
- Rollback: Keep commits atomic; allow easy revert if acceptance criteria aren’t
  met.
- Data Handling: Avoid including secrets in prompts; agents should read envs
  securely and avoid leaking outputs.

Practical Scenarios

- Feature: “Add pagination to the API list endpoint, update frontend, add tests,
  and open a PR.” → Agent.
- Refactor: “Extract `dateUtils` to a shared module and update imports across
  `src/*`; no behavior change.” → Agent.
- Migration: “Replace `request` with `undici` and update all call sites; add a
  wrapper for retries; ship a changelog.” → Agent.
- Q&A Debugging: “Why is `fetch` returning 422? Suggest logs and checks based on
  this stack trace.” → Ask.
- Scoped Edit: “In `src/user.ts`, convert callback code to async/await in
  `getUser()`; keep behavior identical.” → Edit.

Acceptance Criteria

- Read time under 5 minutes; main decisions possible in under 30 seconds.
- Templates are copyable as-is; comparison fits on one screen.
- Each mode has at least two concrete scenarios.
- Safety guidance covers commands, network actions, PRs, and review.
- Clear steps for scoping agents and approving plans.

Links

- Ask vs Edit: see [ai/ai-code-assistants](ai/ai-code-assistants) for related
  notes.
- VS Code workflows: see [README.md](README.md) and tooling notes across this
  repo.
- Diff/Patch workflows: see
  [git-pre-push-generate-codeship-badge.sh](git-pre-push-generate-codeship-badge.sh).

Formatting Cues

- Best Practice: Call out scoping agents and approving plans before commands.
- Warning: Large, multi-file changes should ship via PR with review.
- Tip: Prefer Edit for deterministic single-file changes to keep diffs tight.

Quick Decision Checklist

- Many files or external commands? Agent.
- Need only an explanation/snippet? Ask.
- Exact, small code change in one place? Edit.
- Want progress updates and verification steps? Agent.
- Unsure and timeboxed? Start with Ask, escalate to Agent.

FAQ

- Can an Agent do single-file edits? Yes, but prefer Edit for small,
  deterministic changes to keep review tight.
- When does Ask suffice? When you need insight, examples, or debugging direction
  without changing code.
- How do I stop an Agent? Use the UI to cancel; they provide progress notes and
  can run in background.

Glossary

- Plan/TODOs: Structured steps Agents track and update.
- Background task: Long-running process Agents start with progress updates.
- Approval: Your confirmation before risky actions (commands/network/PR).
- Diff/Patch: The concrete code changes for review.

Links

- Related notes and how-tos: see repo sections on agents, edits, and VS Code
  workflows.

---
> Source: [eoinkelly/notes](https://github.com/eoinkelly/notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
