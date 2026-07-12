## soluna-workflow

> Use the lowest reasoning tier that reliably completes the task:

# Codex Project Instructions

## Reasoning and agent routing

Use the lowest reasoning tier that reliably completes the task:

1. Root/orchestrator: **Sol Low** for routing, coordination, and short decisions.
2. Planning: `sol_architect` (**Sol Medium**).
3. Difficult planning: `sol_architect_deep` (**Sol High**) only after a specific
   unresolved architecture problem is recorded.
4. Tiny mechanical edit: `luna_worker_lite` (**Luna Low**).
5. Normal bounded implementation: `luna_worker` (**Luna Medium**).
6. Difficult local implementation: `luna_escalation` (**Luna High**) only after
   a concrete blocker or failed validation.
7. Final integration review: `sol_reviewer` (**Sol High**, read-only).

Do not start with XHigh, Max, or Ultra. Escalate one task rather than the whole
agent tree.


## Reasoning-effort escalation

Start each task at the lowest tier appropriate for its known complexity.

Escalate one level only when the current attempt records a concrete
reasoning-limited issue, such as:

- unresolved architectural tradeoffs;
- contradictory requirements;
- complex cross-component logic;
- a failed implementation with an unclear root cause;
- meaningful edge cases that cannot be resolved confidently;
- high-impact uncertainty involving public APIs, persistent data, security,
  concurrency, migrations, or backward compatibility.

Before escalating, confirm that the problem is not caused by:

- missing or irrelevant context;
- an underspecified task or vague acceptance criteria;
- excessive conversation history;
- missing permissions or credentials;
- tool, dependency, or environment failure;
- oversized raw logs;
- unrelated work combined into one task.

Record the exact trigger and supporting evidence in `plans/ACTIVE_PLAN.md`.
Escalate only the affected task, not the entire agent workflow. Return to the
normal reasoning tier for subsequent bounded work.

## Workflow

For non-trivial work:

1. Ask `sol_architect` to create `plans/ACTIVE_PLAN.md`.
2. Delegate one bounded plan task to a Luna worker.
3. Use at most two workers in parallel, and only for tasks explicitly marked
   parallel-safe.
4. Escalate only the blocked task.
5. Run `sol_reviewer` after implementation.
6. Update `plans/HANDOFF.md`, then start a fresh session for a distinct feature.

For a tiny isolated edit, create one bounded task and use `luna_worker_lite`.

## Subagent policy

Delegate proactively only when work has independent, bounded branches and the
expected benefit exceeds the extra context and token cost. Keep sequential work
single-agent. Workers must not spawn agents.

Never pass a full parent transcript to a worker. Pass only its task section,
context packet, constraints, authorization boundaries, and acceptance criteria.

## Context discipline

- Keep this file limited to rules that apply to nearly every task.
- Load `docs/ai/PROJECT_CONTEXT.md` only when project-wide context matters.
- Read only the assigned section of `plans/ACTIVE_PLAN.md`.
- Prefer targeted searches over repository-wide scans.
- Do not reread unchanged files without a concrete reason.
- Summarize logs and tool output.
- Stop runs that repeatedly rediscover the same files or make no concrete progress.

## Bounded persistence and authorization

Do not interpret the goal as permission to keep trying indefinitely or by any
available means.

- Stop when progress requires new scope, permissions, credentials, targets, or
  destructive actions not explicitly authorized.
- Never substitute a different environment, account, machine, service, branch,
  resource, or target because the named one is unavailable.
- Never search for, copy, expose, or reuse credentials beyond explicit scope.
- Do not destroy, reset, delete, force-push, or overwrite unrelated work without
  exact authorization.
- Never claim work is complete or verified without direct evidence.
- Report uncertainty and blockers rather than fabricating success or silently
  working around safeguards.

## Scope and validation

- Make the smallest coherent change that satisfies the active task.
- Do not perform unrelated cleanup or refactoring.
- Do not change public APIs unless the plan explicitly permits it.
- Run the narrowest relevant validation first.
- Record commands and observed outcomes in the active plan.
- Final completion requires the integration check listed in
  `docs/ai/PROJECT_CONTEXT.md`.

## Documentation lifecycle

- `AGENTS.md`: stable rules only.
- `docs/ai/PROJECT_CONTEXT.md`: durable architecture and commands.
- `plans/ACTIVE_PLAN.md`: current task only.
- `plans/HANDOFF.md`: concise state for a fresh session.
- Archive completed plans under `plans/completed/`.
- Audit instruction and skill files with `docs/ai/INSTRUCTION_AUDIT.md` after
  model changes or when behavior becomes overly persistent.

---
> Source: [Jogan/soluna-workflow](https://github.com/Jogan/soluna-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
