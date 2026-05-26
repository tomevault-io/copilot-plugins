## agent-safety-harness

> Copy this rule package to a project root to enable it:

# Standard AI Engineering Project Rules

Copy this rule package to a project root to enable it:

```text
AGENTS.md
codex-multi-agent-safe-collaboration.md
skills/
templates/docs/ -> docs/
```

These rules are operating rules for Codex/agent engineering work. They are designed to prevent the model failure modes that matter most: premature completion claims, lazy verification, eager edits before understanding, context rot, scope creep, repeated mistakes, and unsafe multi-agent work.

This source package stores runtime document templates under `templates/docs/`. In an actual project, those templates are instantiated as project-owned `docs/**` files. When updating an existing project from this rule package, do not overwrite existing project `docs/**` runtime files unless the user explicitly asks to reset or regenerate them.

When working on this standard rule source package itself, do not create or update runtime state under `docs/**`. Source-package development plans, design notes, and migration plans belong under repo-root `plans/**`. The `templates/docs/**` tree is only the template source for installed projects.

---

## Scope And Confirmation Rules

- Project coverage: this `AGENTS.md`, the local `./skills/` library, `codex-multi-agent-safe-collaboration.md`, and the explicit control/audit files listed below apply to the project directory where they are copied or placed.
- Default entrypoint: read `skills/using-superpowers/SKILL.md` first. It is the risk router for selecting Fast, Standard, or Strict Path and the required task-specific skills.
- Skill rule: every task must read the skills clearly required by the risk router. A skill is required when the task's success depends on that workflow or its main failure mode applies.
- Read/write scope is mandatory for every task. Before reading broadly or editing anything, identify the allowed read scope and write scope. For pure Q&A, write scope is `None`.
- Default protocol read access: `AGENTS.md`, `codex-multi-agent-safe-collaboration.md`, `./skills/**`, and explicit control/audit files are always allowed as read-only protocol files for the main agent and all subagents. Reading these files is not scope expansion.
- Explicit control files: `docs/current-state.md`, `docs/requirements-matrix.md`, `docs/task-queue.md`, `docs/decisions.md`, `docs/open-questions.md`, `docs/context-checkpoints.md`, and `docs/sprint-contract.md`.
- Explicit audit/harness files: `docs/agent-mistake-ledger.md`, `docs/tooling-and-mcp-registry.md`, and `docs/evidence/**`.
- Non-control docs: `docs/plans/**`, `docs/platform/**`, `docs/agent-work-summary.md`, and any other `docs/**` files are not default control files. Read them only when the task package, user request, or recovery need explicitly authorizes them. `docs/agent-work-summary.md` may be read when historical evidence is needed.
- User-requested process skips: if the user asks to skip skills, TDD, review, browser verification, context-rot protection, recovery, mistake-ledger recording, or other required protocol, restate the requested skip and ask for one explicit confirmation before doing work.
- Protocol precedence: if a user request, task target, shortcut, blocker, or proposed skip conflicts with this file, a required `SKILL.md`, or `codex-multi-agent-safe-collaboration.md`, follow the protocol first unless the protocol itself requires user confirmation or continuing would risk destructive, irreversible, or security-sensitive changes.
- Commit confirmation rule: `git add` / `git commit` may happen only at the end of the current turn or execution phase, and only after asking the user once. Only commit after the user explicitly confirms. Subagents must not run `git add` or `git commit`.
- Git/worktree availability: if the current directory is not a Git repository, or worktrees are unavailable, do not force Git operations. Use the nearest safe file-based workflow, report the limitation, and keep changes uncommitted.

---

## Universal Gates

These gates apply on every path, including Fast Path.

- **No false completion:** do not claim work is complete, fixed, passing, ready, or accepted without fresh verification evidence. If verification has not run, say what changed and what remains unverified.
- **No eager edits:** before changing files, read the relevant context and identify the read/write scope. For bugs, failures, or unexpected behavior, investigate root cause before fixing.
- **No scope creep:** edit only the authorized write scope. If the needed scope expands, stop and state the new scope before proceeding.
- **No automatic commits:** do not stage or commit unless the user explicitly confirms.
- **Interrupted work recovers first:** after interruption, context loss/compaction, tool-session loss, crash, timeout, or long pause on Standard or Strict work, perform recovery before writing code or claiming status.
- **Mistakes become prevention:** record wrong root causes, wrong fixes, repeated failures, missed verification with a success claim, material user corrections, regressions, and scope violations. Record other mistakes when they are likely to recur or cause real damage; do not record harmless spelling/style corrections unless they repeat.

---

## Risk Router

Choose the lightest path that still satisfies the Universal Gates and task risk.

### Fast Path

Use for:

- Q&A, explanation, summarization, small documentation advice, simple inventory, or low-risk non-behavioral single-point edits.
- Small config or text changes with no behavior change and no cross-file contract.
- Single-point code edits are Fast Path only when they do not change business behavior, public contracts, security/data/deployment behavior, or cross-file assumptions, and can be directly verified.

Rules:

- Read `skills/using-superpowers/SKILL.md`; read another skill only when clearly required by the trigger table below.
- Mandatory read/write scope still applies.
- No control-file update is required unless the task directly edits those files.
- Before a Fast Path completion claim, inspect the changed line or diff, confirm the write scope stayed narrow, and run the smallest direct check when one exists. If no command applies, report file/diff inspection instead of implying tests passed.
- Completion claims still require fresh evidence.

### Standard Path

Use for:

- Normal feature work, bugfixes, refactors, behavior changes, test changes, and user-facing UI changes.
- Work that touches a few files but does not require Strict Path controls.

Rules:

- Read the task-specific skills required by the trigger table.
- Mandatory read/write scope applies before edits and before any subtask handoff.
- For bugfixes and failures, use `systematic-debugging`.
- For TDD-required categories, use `test-driven-development` test-first. Verify-after is allowed only for copy/text, styling/layout-only UI, low-risk configuration, mechanical formatting/renames, throwaway/generated code, or cases where no automated test is feasible; record the reason and run the strongest available verification.
- UI, layout, responsive, and browser behavior changes require `ui-browser-verification`, but do not automatically upgrade to Strict Path.
- Use Standard Recovery after interruption before continuing.
- Update only docs/control files that are actually affected by the work.

### Strict Path

Use for:

- Cross-module behavior, public API, database/schema, migrations, auth, permissions, security, payment, deployment, or data-integrity changes.
- Long-running work, cross-session continuation, multi-agent work, interrupted Strict work, production/online bugs, failed-fix retries, or repeated model mistakes.
- Ambiguous requirements that need a contract before implementation.
- UI work that is cross-module, state-heavy, auth/payment/security-related, production-observed, or already failed verification.

Rules:

- Read `codex-multi-agent-safe-collaboration.md` when multi-agent or cross-module coordination is involved.
- Use `docs/sprint-contract.md`, affected control files, durable evidence where useful, mistake-ledger checks when relevant, and evaluator acceptance before final completion.
- Use Strict Recovery after interruption.
- Store durable evidence under `docs/evidence/**` for important claims, UI verification, bug root cause, failed hypotheses, multi-agent integration, and recovery.

---

## Skill Trigger Table

Read a skill when the current task clearly matches its trigger. "Clearly" means the task's success depends on that workflow, the task directly asks for it, or the workflow's failure mode is present.

| Skill | Required When |
| --- | --- |
| `using-superpowers` | Starting any task in this project; use it as the risk router. |
| `brainstorming` | New or ambiguous feature/product/design work where intent, options, or acceptance need clarification before implementation. |
| `writing-plans` | A multi-step implementation plan is needed before touching code, especially when work spans files, phases, or handoffs. |
| `executing-plans` | Executing an existing written plan in a single-agent flow with checkpoints. |
| `subagent-driven-development` | Main-agent-approved task dispatch inside the multi-agent protocol. |
| `dispatching-parallel-agents` | Two or more independent investigations or tasks can safely proceed in parallel inside the multi-agent protocol. |
| `test-driven-development` | New or changed business behavior, bugfix regression protection, public/API behavior, state flow, data transformation, permission/security/payment logic, or risky refactor. |
| `systematic-debugging` | Any bug, failing test/build, runtime error, unexpected behavior, production issue, or failed prior fix before proposing fixes. |
| `learning-from-mistakes` | Wrong root cause, wrong fix, repeated failure, missed verification with a success claim, material user correction, regression, or scope violation. |
| `ui-browser-verification` | Any frontend, UI, visual, layout, responsive, browser interaction, or web app behavior completion claim. |
| `verification-before-completion` | Before claiming work is complete/fixed/passing/ready, before PR/merge/commit completion, or before closing a task. |
| `evaluator-acceptance-review` | Before declaring Strict Path, long-running, multi-agent, cross-module, risky, high-impact, or production-facing work complete. |
| `requesting-code-review` | After major implementation, complex fixes, Strict integration, or when a fresh review materially reduces risk. |
| `receiving-code-review` | Before acting on user or reviewer feedback that changes code, scope, architecture, or tests. |
| `using-git-worktrees` | Feature work needs isolation and Git/worktrees are available. |
| `finishing-a-development-branch` | Implementation is complete and verified, and the user wants merge/PR/branch cleanup decisions. |
| `writing-skills` | Creating or modifying skills. |

If multiple skills are required, read only the directly relevant ones. Process skills come before implementation skills.

---

## Context Rot And Recovery

Do not rely on chat context as the only source of truth for Standard or Strict work after interruption.

### Standard Recovery

Trigger Standard Recovery after interruption, context loss/compaction, tool-session loss, crash, timeout, or long pause during Standard Path work.

Before continuing implementation:

1. Stay read-only.
2. Re-read the user goal or latest task instruction, `AGENTS.md`, `skills/using-superpowers/SKILL.md`, and the task-specific skills.
3. Inspect current file state: changed files, relevant diffs or file contents, running processes/logs when relevant, and unverified work.
4. Restate read/write scope.
5. Identify what is complete, what is partially done, what is unverified, blockers, and the next safe action.
6. Only then continue writing.

### Strict Recovery

Trigger Strict Recovery for interrupted Strict Path, long-running, cross-session, production, multi-agent, or failed-fix work.

Before continuing implementation, read in order:

1. `AGENTS.md`
2. `codex-multi-agent-safe-collaboration.md` when multi-agent/cross-module coordination is active
3. Relevant `skills/<name>/SKILL.md`
4. `docs/current-state.md`
5. `docs/requirements-matrix.md`
6. `docs/task-queue.md`
7. `docs/decisions.md`
8. `docs/open-questions.md`
9. `docs/context-checkpoints.md`
10. `docs/sprint-contract.md`
11. `docs/tooling-and-mcp-registry.md` when selecting verification/debugging tools
12. `docs/agent-mistake-ledger.md` when debugging retries or mistakes may apply
13. `docs/agent-work-summary.md` when historical evidence is needed

Strict Recovery must complete before new writes, new implementation agents, commits, merges, or completion claims.

Checkpoint rule for Strict Path: add or update a context checkpoint every 60-90 minutes of active long-running work, every 2-3 completed tasks, after interruption recovery, and before cross-session continuation.

---

## Control File Ownership

Avoid duplicated source-of-truth facts. Each file owns one category of truth; other files should reference IDs or links instead of copying full details.

- `docs/current-state.md`: short current summary, verified status, blockers, risks, and next safe task links.
- `docs/sprint-contract.md`: Strict Path goal, non-goals, allowed/forbidden scope, acceptance contract, and completion conditions.
- `docs/requirements-matrix.md`: requirement IDs, status, acceptance evidence links, tests/checks, and risks.
- `docs/task-queue.md`: dispatch-ready task packages, read/write scope, task state, and verification commands.
- `docs/decisions.md`: confirmed product, architecture, security, data, workflow, or rollout decisions and revisit triggers.
- `docs/open-questions.md`: unresolved questions, blockers, owners, and conservative defaults.
- `docs/context-checkpoints.md`: recovery/checkpoint snapshots, drift detection, next tasks, and residual concerns.
- `docs/tooling-and-mcp-registry.md`: default tool choices and required evidence by task type.
- `docs/agent-mistake-ledger.md`: meaningful agent mistakes and prevention mechanisms.
- `docs/evidence/**`: durable proof behind important claims.
- `docs/agent-work-summary.md`: reviewed multi-agent or historical integration summaries, not the current status entrypoint.

Completion gate for Standard/Strict work: update every affected owner file. Do not update unrelated control files just to satisfy a checklist. If a required owner file is stale relative to the work, synchronize it or record why no update is needed before claiming completion.

---

## Standard Development Workflow

### New Work

1. Read `skills/using-superpowers/SKILL.md` and classify Fast, Standard, or Strict.
2. Declare read/write scope.
3. Read task-specific skills required by the trigger table.
4. Understand relevant project state by reading only the needed docs/code.
5. For Strict Path, write or update `docs/sprint-contract.md` before implementation when scope/acceptance could drift.
6. For large or multi-agent work, use the multi-agent protocol or a simulated equivalent with bounded task packages.
7. Implement within scope.
8. Run path-appropriate verification and browser/tool checks when required.
9. Update only affected control/audit files.
10. Ask before any Git commit.

### Bug Or Failure

1. Read `systematic-debugging`.
2. Check `docs/agent-mistake-ledger.md` when there was a prior failure, repeated issue, material correction, or Strict Path risk.
3. Reproduce or gather evidence for the failure.
4. Find root cause before editing.
5. Add failing regression protection first where the change is TDD-required; use verify-after only for an allowed exception and record why.
6. Implement the minimal fix.
7. Verify with fresh commands and tool evidence.
8. If the agent made a mandatory-record mistake or another mistake likely to recur or cause real damage, use `learning-from-mistakes` and update `docs/agent-mistake-ledger.md`.

### Completion

Before claiming completion:

1. Read `verification-before-completion`.
2. Run relevant tests/type checks/lint/build/format or state exact blockers.
3. For frontend/UI work, read `ui-browser-verification` and verify in Chrome DevTools MCP or an equivalent browser harness.
4. For Strict Path, use `evaluator-acceptance-review`.
5. Check whether a mistake-ledger entry is required.
6. Update affected owner files only.
7. Report evidence, unverified gaps, and residual risk.
8. Ask before any Git commit.

---
> Source: [Djh0311/agent-safety-harness](https://github.com/Djh0311/agent-safety-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
