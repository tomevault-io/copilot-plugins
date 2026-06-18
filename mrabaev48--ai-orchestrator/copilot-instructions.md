## ai-orchestrator

> This repository is a TypeScript monorepo for AI and orchestration-oriented packages.

# AGENTS.md

## Project scope

This repository is a TypeScript monorepo for AI and orchestration-oriented packages.

Treat this codebase as production-facing infrastructure. Optimize for:
- correctness
- determinism
- debuggability
- explicit contracts
- operational safety
- small, reviewable changes

Assume the repository uses:
- `pnpm`
- `turbo`
- `vitest`
- `eslint`
- `tsc`

Unless the task explicitly requires broader changes, work only within the relevant orchestration package and its immediate dependencies.

---

## Primary operating principles

- First understand the current codepath before editing.
- Prefer the smallest safe change that fully solves the task.
- Preserve architectural boundaries.
- Preserve public contracts unless the user explicitly authorizes a breaking change.
- Do not perform unrelated refactors.
- Do not add dependencies unless there is no reasonable solution with the existing stack.
- Do not modify adapters when the task is confined to the domain layer unless the design makes it necessary.
- Prefer explicit types, narrow interfaces, and structured results over implicit object shapes.
- Do not hide uncertainty. If something is blocked, ambiguous, or out of scope, say so directly.

---

## Repository assumptions

Treat the repository as having these logical layers unless the code clearly demonstrates otherwise:

1. **Domain / orchestration core**
   - workflow logic
   - routing
   - planning
   - execution flow
   - state transitions
   - policy decisions

2. **Adapters / integrations**
   - model providers
   - tool providers
   - transport adapters
   - persistence connectors
   - queue or messaging integrations

3. **Interface / API / app surface**
   - package entrypoints
   - configuration surfaces
   - externally consumed types
   - events, payloads, schemas

Keep those boundaries clean.

---

## Non-negotiable architecture rules

### Boundary rules
- Keep orchestration and domain logic separate from provider-specific logic.
- Keep transport concerns out of domain flow unless the architecture explicitly centralizes them there.
- Keep persistence details out of decision-making logic unless persistence is the explicit source of truth.
- Do not leak raw provider responses into higher-level orchestration code without normalization.
- Do not introduce implicit coupling across packages when a typed interface already exists or should exist.

### Contract rules
- Preserve public APIs and public types unless the task explicitly allows breaking changes.
- Preserve config compatibility unless migration is explicitly in scope.
- Preserve existing event and payload shapes unless a change is explicitly approved and documented.
- Prefer additive changes over breaking changes.
- When changing internal contracts, keep the change localized and typed.

### Refactor rules
- Do not combine feature work with unrelated cleanup.
- Do not rename broad surfaces just because a name could be better.
- Do not move files or modules unless it materially improves correctness or is required by the task.
- If architectural debt is discovered, separate it from the requested implementation unless the user asked for deeper restructuring.

---

## AI orchestration invariants

For any task involving orchestration or execution flow, treat these as core invariants:

- deterministic-first behavior
- explicit retry semantics
- explicit timeout behavior
- explicit cancellation behavior
- idempotency under retry or replay where applicable
- explicit state transitions
- structured error propagation
- normalized tool contracts
- observability through logs, traces, or correlated identifiers

Do not treat these as optional polish.

---

## Execution safety rules

Whenever a task touches execution flow, explicitly inspect the following:

### Retries
- Are retries explicit and bounded?
- Can retry duplicate external side effects?
- Is retry state isolated between attempts?
- Are retry decisions visible in code and diagnosable in failures?

### Timeouts
- Is timeout enforced at the right boundary?
- Is timeout surfaced clearly to callers?
- Can timeout leave hidden work running?
- Is retry-after-timeout behavior safe?

### Cancellation
- Can cancellation interrupt the system mid-transition?
- Can cancellation leave state inconsistent?
- Can cancellation strand side effects, locks, or queued work?

### Idempotency
- Could re-entry, replay, retry, or race conditions execute the same step twice?
- Are non-idempotent side effects protected by deduplication or state guards?

### State integrity
- Are transitions explicit and reconstructable?
- Are illegal or ambiguous transitions possible?
- Is failure recovery deterministic or at least diagnosable?

### Tool contracts
- Are tool inputs typed and validated?
- Are tool outputs normalized before orchestration consumes them?
- Are tool errors structured enough for callers and operators?

### Observability
- Can operators reconstruct a failed path from logs, traces, and identifiers?
- Are important transitions and decisions visible?
- Do errors preserve enough context to debug the incident?

If a change materially affects these areas, review them before declaring the task complete.

---

## Default implementation workflow

Unless the user explicitly asks for a different flow, use this sequence:

1. Identify the target package and the concrete behavior to change.
2. Inspect the relevant codepath before editing.
3. Summarize the current design briefly.
4. Produce a short plan tied to concrete files or modules.
5. Implement the smallest safe change.
6. Update or add tests when the change is non-trivial.
7. Run validation.
8. Review git status.
9. If requested, commit, push, and open a draft PR.
10. Report results in the required output format.

Do not skip understanding the existing codepath before making changes.

---

## Validation policy

Use Turbo for repository validation.

Run these commands unless they are clearly irrelevant or blocked by known out-of-scope failures:

```bash
turbo run lint
turbo run test
turbo run typecheck
turbo run build
```

##Validation rules

* Do not claim validation passed unless the commands actually ran.
* If a command fails, report the exact failing command.
* If validation is blocked by an existing or out-of-scope issue, stop and report the blocker clearly.
* Do not silently widen scope to fix unrelated failures.
* If you run narrower checks first for iteration speed, still report that they were narrower than the standard repo-wide validation.

##Test expectations

For non-trivial changes, prefer to cover:

* success path
* failure path
* regression path
* retry / timeout / cancellation path if execution flow is touched

Avoid redundant tests, but do not leave critical risk areas untested.

⸻

##Git workflow

##General rules

* Inspect git status before finalizing.
* Do not commit unless the user asked for it or the task explicitly includes commit/publish.
* Use conventional commits.
* Keep commit scope specific to the changed package or subsystem.
* Do not claim a push or PR exists unless it was actually confirmed.

##Conventional commit format

Use:

```
type(scope): brief description
```

## Examples:

* feat(orchestrator): add cancellation-safe retry guard
* fix(dispatcher): prevent duplicate execution on retry
* refactor(state): isolate transition validation logic
* test(workflow): add retry timeout regression coverage

## Publish workflow

If the user asks to publish:

1. Check the current branch.
2. Push the branch with upstream configured if needed.
3. Open a draft PR by default.
4. Report:
    * current branch
    * upstream status
    * push status
    * PR status

If push or PR creation is blocked, report the exact blocker instead of implying completion.

⸻

## Scope control and blockers

If you encounter failures or repository issues outside the requested scope:

* stop
* describe the blocker precisely
* explain why it is outside scope
* avoid unrelated cleanup unless the user explicitly expands the task

## Examples of out-of-scope blockers:

* unrelated package build failures
* pre-existing repo-wide lint failures in untouched modules
* missing environment or credentials for publication
* unrelated adapter breakage during a domain-layer task

## Be precise about what was changed versus what remains blocked.

⸻

## Code quality expectations

## Prefer

* small patches
* explicit control flow
* typed interfaces
* structured error shapes
* narrow helper abstractions
* clear naming
* localized changes
* additive compatibility

## Avoid

* broad speculative abstractions
* hidden side effects
* implicit shared mutable state
* ad hoc object shapes where typed contracts are expected
* provider-specific leakage into orchestration core
* mixing feature work with unrelated cleanup
* dependency additions for convenience alone

⸻

## Review guidance

When reviewing or explaining a change, distinguish clearly between:

Confirmed issues

Problems that are directly supported by the code, tests, or validation output.

Credible risks

Concerns that are strongly suggested by the implementation but not fully proven by runtime evidence.

Optional improvements

Non-blocking ideas that may improve maintainability, clarity, or resilience.

Do not blur those categories.

⸻

## Required response format

## Unless the user explicitly asks for a different format, always respond in this structure:

1. Thinking/Understanding
2. Plan
3. Changes
4. Validation
5. Risks
6. Git status

## Format rules

* Keep it concise but specific.
* Mention concrete files, modules, or packages where relevant.
* For pure review tasks with no code changes, say:
    * Changes: no code changes made
    * Git status: unchanged
* For failed validation, include the exact failing command and the blocker summary.
* For skipped validation, say what was skipped and why.

⸻

## Truthfulness rules

* Do not invent files, modules, packages, commands, tests, or PRs.
* Do not say a check passed unless it actually passed.
* Do not claim backward compatibility unless the affected contract was actually preserved or reviewed.
* Do not present partial work as complete.
* Do not mask uncertainty with vague language.

If something is unknown, blocked, or not verified, state that explicitly.

⸻

## Task routing guidance

## Use implementation mode for:

* implementing a feature
* fixing a bug
* modifying orchestration behavior
* adding or updating tests
* preparing code for commit, push, or PR

## Use execution-safety review mode for:

* retry analysis
* timeout or cancellation review
* duplicate execution risk
* partial failure safety
* idempotency concerns
* runtime incident analysis

## Use architecture/review mode for:

* architecture drift review
* boundary audits
* contract review
* state transition review
* reviewer hotspot analysis

## Use PR-writing mode for:

* drafting PR descriptions
* summarizing validation
* calling out risks and reviewer focus

⸻

## Final completion checklist

Before declaring work complete, verify:

* the requested scope was addressed
* the current codepath was inspected before editing
* architectural boundaries were respected
* public contracts were preserved unless explicitly allowed to change
* tests were added or updated when appropriate
* validation results were reported honestly
* git status was checked
* publish status was reported if requested
* risks and blockers were stated clearly

## If any of the above is missing, do not describe the task as fully complete.

---
> Source: [mrabaev48/ai-orchestrator](https://github.com/mrabaev48/ai-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
