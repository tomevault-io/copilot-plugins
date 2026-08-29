## agent-loop-skill

> Use for long-running, multi-step, verifiable, restartable AI agent tasks that need contracts, disk state, traces, role separation, subjective scoring, or cross-session continuity.

# Agent Loop Skill

Use this rule when a task needs multiple execution rounds, cross-file changes, tests, browser checks, subjective review, batch generation, recovery after failures, or work across sessions.

Skip it for simple questions, one-line edits, typo fixes, and single commands.

## Core Loop

```text
Understand -> Contract -> Role split -> Write state -> Verify -> Read traces -> Fix or restart -> Ship -> Clean harness
```

## Required Disk State

Create `.agent-loop/` in the target project:

```text
.agent-loop/contract.md
.agent-loop/progress.md
.agent-loop/log.md
```

Use optional files only when useful:

```text
.agent-loop/state.json
.agent-loop/scorecard.md
.agent-loop/defects.md
.agent-loop/traces/
```

If the state cannot fit into the core files, split the task.

## Contract First

Before generating final code or content, write `.agent-loop/contract.md` with:

```text
Goal:
In scope:
Out of scope:
Definition of done:
Acceptance criteria:
Validation commands:
Subjective scoring criteria:
Risk assumptions:
Stop conditions:
Restart conditions:
Final deliverables:
```

Every acceptance item must be verifiable by command output, screenshot, log, diff, checklist, or scorecard.

## Separate Roles

- Planner writes the contract and risks.
- Generator implements inside the contract.
- Evaluator runs checks, reads traces, and decides pass/fail.
- Referee arbitrates only when needed.

Do not let the Generator lower acceptance criteria for its own work.

## Read Traces

Use evidence before judgment:

- tests
- build logs
- console errors
- browser screenshots
- API responses
- diffs
- generated artifacts

If evidence conflicts with intuition, trust the evidence.

## Restart Criteria

Restart from the contract when:

- the same acceptance item fails twice
- patches are accumulating
- the output passes tests but misses the user goal
- the log shows the contract was forgotten
- a clean retry is cheaper than continuing

Append the failure reason to `.agent-loop/log.md` before restarting.

## Score Subjective Work

For design, writing, UX, architecture, or taste-heavy work, score:

- Functionality
- Simplicity
- Craft
- Originality

Use 0-10 scores with one sentence of evidence per dimension.

## Completion

Only finish when the contract passes, evidence exists, blockers are resolved, logs explain the path, subjective work is scored or approved, and temporary harness files are removed or justified.

---
> Source: [dososo/agent-loop-skill](https://github.com/dososo/agent-loop-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
