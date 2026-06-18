## 10ximplement

> Use when the user wants a long-running implementation run with phase gates, durable progress tracking, subagent execution, review checkpoints, or explicit wording like "$10ximplement", "10x implement", "run this plan", "phase-gate this build", "orchestrate builders", "keep implementation notes", "track decisions", or "continue this implementation". Combines a goal ledger, PM phase gates, subagent-driven plan execution, and executing-plans fallback into one workflow.


# 10ximplement

Use this skill to run implementation like an orchestrated build: keep durable state, choose narrow phases, execute through builders or direct plan steps, review before each gate, and leave the next agent a clear resume point.

## Core Contract

- Keep `.agent/runs/<goal-id>/implementation-notes.html` as the canonical current truth for the run.
- Keep `.agent/runs/<goal-id>/GOAL.md` as the contract: objective, finishing criteria, parent goal, native goal coupling line if relevant, and escape hatch.
- Keep `.agent/GOALS.md` as the project-level run index.
- Use `assets/implementation-notes.template.html` and `scripts/init_10x_ledger.py` when starting a new run.
- Update the HTML after phase gates, validation, blockers, accepted defers, handoffs, meaningful implementation checkpoints, and final completion.
- Ask at phase boundaries when the user must choose PM or review mode. Once a plan execution mode is chosen, execute continuously until blocked, all tasks complete, or the gate requires a decision.

## Start Or Resume

1. Inspect current context: user request, repo docs/specs, latest handoff or existing `.agent/` run, and `git status --short`.
2. If no ledger exists for this run, create one:

```bash
python3 <skill-dir>/scripts/init_10x_ledger.py \
  --root . \
  --goal-id "<stable-goal-id>" \
  --title "<short title>" \
  --objective "<one sentence objective>" \
  --mode full
```

3. If a ledger already exists, read `GOAL.md` and the top `Resume Here` section before touching code.
4. Only create or update a native runtime goal when the user explicitly asks for goal mode, `$goal`, `/goal mode`, "start a goal", or when a native runtime goal already exists. Put the ledger path in that runtime goal objective.
5. Define or confirm finishing criteria before implementation.

## Phase Gate Loop

Repeat this loop until the goal is complete:

1. **Load context:** read the canonical product/spec entrypoint, latest memory/handoff, current build spine, only relevant domain docs, and changed files.
2. **Review previous phase:** verify what was supposed to be true, what exists, what checks pass, and which issues are blockers versus defers.
3. **Choose next slice:** pick one narrow vertical phase with product value and stable architecture. Name non-goals. Avoid mixing unrelated infrastructure, billing, provider, messaging, and UI polish unless the phase explicitly requires it.
4. **Select PM mode if not already chosen for this phase:**
   - Traditional PM Mode: write a compact builder prompt or phase brief for external implementation. Do not edit code directly.
   - Inline PM Mode: run the phase in this chat. Make only small low-risk edits yourself; use builder subagents for larger, risky, or multi-file work.
5. **Prepare the phase brief:** include working directory, files/docs to read, build scope, non-goals, required checks from existing project scripts, stop condition, and expected summary.
6. **Execute or hand off:** use the execution mode below.
7. **Review gate:** run the selected review mode, verify findings, run checks, decide accept checkpoint, request fixes, shrink scope, or redirect.
8. **Ledger update:** record status, decisions, logic, validation, blockers, accepted defers, protected paths, evidence links, and next exact action in `implementation-notes.html`.

## Execution Modes

Use the mode that fits the phase:

- **Subagent-driven execution:** Use when a written plan exists, tasks are mostly independent, and subagents are available in the current session. Dispatch a fresh implementer per task, then spec compliance review, then code quality review. Do not move to the next task while either review has open issues.
- **Executing-plans fallback:** Use when a written plan exists but subagents are unavailable, tasks are tightly coupled, or execution is happening as a direct sequential session. Read the plan, review it critically, follow every task exactly, run stated verification, and stop on blockers or repeated verification failures.
- **PM handoff:** Use in Traditional PM Mode. Write the builder prompt/phase brief and wait for returned builder output before reviewing.
- **Small direct edit:** Use in Inline PM Mode only for narrow, low-risk changes. Still review and update the ledger.

Before executing implementation plans, respect branch safety: do not start implementation on `main` or `master` without explicit user consent. Prefer an existing isolated worktree; do not fight a platform-managed workspace.

## Subagent Rules

When using subagents:

- Provide full task text and curated context. Do not make subagents read the whole plan to discover their task.
- Use one implementer per task unless a PM phase has clean, non-overlapping builder ownership lanes.
- In Codex, keep at most six subagents open at a time. "Open" includes stale completed subagents, not only running ones. Close out subagents as soon as their result has been captured, verified, or superseded so the controller can launch the next needed agent.
- For each task, run review in this order: spec compliance first, code quality second.
- Reviewer findings must be fixed and re-reviewed before proceeding.
- If an implementer reports `NEEDS_CONTEXT`, provide missing context and re-dispatch.
- If an implementer reports `BLOCKED`, change something: provide context, use a more capable model, split the task, or escalate to the user if the plan is wrong.
- Use `references/subagent-prompts.md` when constructing implementer, spec reviewer, and code quality reviewer prompts.

## Review Modes

Ask the user for review mode before starting review unless they already chose or explicitly asked you to use judgment:

- **Normal Review Mode:** three narrowly scoped review agents.
- **Chad Review Mode:** six review agents: three lane-specific reviewers plus three full-integration reviewers.

Before launching review agents, close stale implementer or reviewer subagents whose outputs have already been consumed. Chad Review uses the full six-open-agent budget, so it requires a clean slate unless the platform provides a higher limit.

Choose review lanes from the phase risk, not a fixed checklist. Examples: product/scope alignment, architecture/data correctness, security, UX quality, tests/verification, runtime/deployment safety, billing correctness, provider/secret handling, code maintainability, docs/handoff quality.

As orchestrator, verify reviewer findings by checking cited files/lines and running the needed checks. Do not become an extra full reviewer. Synthesize one gate verdict and one focused correction prompt limited to must-fix and selected should-fix items.

## Decisions And Defers

Record decisions in `implementation-notes.html` when the spec is silent, ambiguous, contradicted by the repo, or when multiple valid paths exist.

Each decision must include:

- **Decision:** what was chosen.
- **Logic:** the reasoning used to choose it. Keep this to one concise sentence for small decisions; use a short paragraph for architecture, product behavior, data integrity, security, UX direction, or phase sequencing decisions.
- **Impact:** what this affects.
- **Revisit if:** the trigger that should make a future agent reconsider.

Record accepted defers durably in the HTML before accepting a checkpoint. Do not let defers silently become next-phase scope.

## Ledger Rules

Keep `Resume Here` short enough to read in under a minute and concrete enough to resume immediately. It should include status, current phase, completed work, active work, blockers, next exact action, last validation, and protected paths.

Use these states:

- `[todo]` known and not started
- `[doing]` currently active
- `[done]` completed and validated
- `[blocked]` waiting on external input or dependency
- `[incomplete]` attempted, not fully solvable inside current constraints
- `[abandoned]` intentionally stopped because the goal changed or no longer pays rent

Use `[incomplete]` only with reason, proof, attempted work, impact, and next human/agent decision.

Append a compact object to the inline `progressEvents` array for every checkpoint. Save bulky proof under `evidence/` and link it from the HTML.

## Stop Conditions

Pause, ask the user, mark a scoped item `[blocked]` or `[incomplete]`, or shrink the phase if:

- validation contradicts the goal
- the goal requires a scope change
- the plan has critical gaps
- the agent is looping without measurable progress
- the next step risks deleting or rewriting durable memory
- the PRD and actual repo disagree in a way that affects behavior
- verification fails repeatedly
- native or git worktree actions would discard user work

## Finish

Before reporting completion:

1. Re-read `GOAL.md` finishing criteria.
2. Run required checks or explain why they cannot run.
3. Resolve review gate findings or record accepted defers.
4. Update `implementation-notes.html` with final status, decisions, tradeoffs, validation, blockers, defers, and next-goal candidates.
5. Append a final `progressEvents` entry.
6. Update `.agent/GOALS.md` final status.
7. If a native runtime goal is coupled, mark it complete only when the objective is actually finished and verified.
8. Report the ledger path and verification outcome.

---
> Source: [cobibean/10ximplement](https://github.com/cobibean/10ximplement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
