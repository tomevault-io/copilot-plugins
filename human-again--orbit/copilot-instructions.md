## orbit

> This file gives Codex project-specific instructions for working inside the generic coding-agent harness and its end-to-end demo.

# AGENTS.md

This file gives Codex project-specific instructions for working inside the generic coding-agent harness and its end-to-end demo.

## Purpose

Use this repository as a vendor-neutral harness for long-running coding workflows with planning, bounded research, validation, evaluation, review, retries, checkpoints, and optional LLM-as-judge support.

When operating in this repo, prioritize **harness correctness over feature speed**. A small, verifiable change is better than a broad, fragile change.

## Primary goals

- Keep work bounded to one task per iteration.
- Respect mission files, backlog dependencies, validation gates, and risk controls.
- Produce structured artifacts that the harness can evaluate.
- Prefer deterministic evidence over self-reported success.

## Files to read first

Before making changes, read these in order when they exist:

1. `mission.md`
2. `agent-rules.md`
3. `backlog.json`
4. `progress.md`
5. `runtime/config*.json`
6. `research/<task-id>.md` if the chosen task requires research
7. `docs/phase-*.md` when modifying harness architecture
8. `examples/real-e2e-demo/docs/eval-plan.md` when working on the demo or eval logic

## Task selection rules

- Work on exactly one task at a time.
- Choose the highest-priority unfinished task whose dependencies are complete.
- Do not skip dependency order unless explicitly instructed.
- If the harness already selected a task for the run, do not override it.
- If requirements are ambiguous, stop and mark the task blocked with a precise reason.

## Change scope rules

- Make the smallest coherent change that advances the selected task.
- Do not refactor unrelated modules opportunistically.
- Do not add dependencies unless the task explicitly requires them.
- Do not change mission scope, acceptance criteria, or evaluator thresholds unless asked.
- If a broader cleanup is needed, record it in `progress.md` instead of folding it into the current task.

## Coding rules

- Prefer readable, boring code over clever code.
- Keep functions small and testable.
- Preserve existing file layout unless there is a strong reason to change it.
- Add or update tests whenever behavior changes.
- If creating structured outputs for the harness, use strict JSON with stable keys.
- Avoid introducing hidden state or nondeterminism unless the task explicitly calls for it.

## Validation rules

Run the project’s configured validation commands whenever possible.

Typical validation includes:

- syntax or compile checks,
- unit tests,
- linting,
- type checks,
- harness-specific artifact checks.

Never declare success only from reasoning. Prefer proof from:

- passing validation output,
- changed files,
- diff evidence,
- evaluator artifacts.

If validation fails:

- fix it if the fix is in scope,
- otherwise report the failure precisely,
- do not mask or ignore failing checks.

## Harness-specific rules

### For orchestrator changes

When editing `orchestrator.py` or control-plane files:

- preserve deterministic state transitions,
- keep file writes explicit and easy to inspect,
- avoid patch-on-patch string rewriting patterns,
- prefer direct source edits over generated mutation scripts,
- ensure compileability after each change.

### For evaluator or reviewer changes

- Keep deterministic gates first.
- Treat LLM judging as advisory unless explicitly configured otherwise.
- Require structured outputs for judge/evaluator artifacts.
- Prefer rubric-based scoring over vague natural-language assessments.

### For command execution

- Route shell commands through the risk-gating path when the repo provides one.
- Do not bypass command classification for convenience.
- Treat blocked or high-risk commands as requiring explicit escalation.

### For resume, checkpoint, and budget logic

- Preserve durability and inspectability of state files.
- Avoid silent resets of budget or checkpoint state.
- If a recovery path is incomplete, fail loudly and record the missing behavior.

## Research rules

Use research only when the task needs outside information.

When research is required:

- keep it bounded,
- answer only the explicit questions relevant to the task,
- distill findings into implementation guidance,
- do not dump raw search output into prompts or progress logs.

A research artifact should contain:

- the question,
- key findings,
- sources,
- risks,
- recommendation.

## Progress logging

When completing an iteration, append concise notes to `progress.md` with:

- task id,
- what changed,
- validation status,
- blockers or risks,
- next recommended step.

Keep entries factual and brief.

## Output contract for coding-agent runs

When the harness expects a structured coding-agent result, return JSON in this general shape:

```json
{
  "status": "complete",
  "changed_files": ["path/to/file.py"],
  "notes": "Short summary of what was done and what remains.",
  "metadata": {
    "reasoning_mode": "concise"
  }
}
```

Allowed `status` values:

- `complete`
- `blocked`
- `needs_human`
- `failed`

Do not invent extra status values unless the harness is updated to handle them.

## End-to-end demo guidance

For `examples/real-e2e-demo/`:

- implement tasks in the order defined by `backlog.json`,
- keep to Python standard library only,
- use in-memory state only,
- make tests the main source of proof,
- ensure the outcome can be classified as success, partial success, blocked, or failure by the evaluation plan.

## What good looks like

A strong iteration in this repo usually has all of these properties:

- the chosen task is dependency-correct,
- only relevant files change,
- validation runs and passes or fails clearly,
- evaluator artifacts match the real state of the repo,
- progress notes are updated,
- no risky commands are bypassed,
- no unnecessary scope expansion occurs.

## What to avoid

Avoid these common failure modes:

- claiming completion with no meaningful diff,
- changing unrelated files,
- silently weakening tests,
- skipping validation because the code “looks right”,
- bypassing risk gates,
- patching generated code through fragile string replacements,
- overusing the LLM judge where deterministic checks are available.

## Preferred workflow

1. Read mission, rules, backlog, and progress.
2. Confirm the selected task.
3. Inspect relevant files only.
4. Make the smallest coherent change.
5. Update or add tests.
6. Run validation.
7. Check diffs and artifacts.
8. Update progress.
9. Return a structured result.

## Escalate instead of guessing when

Escalate with `blocked` or `needs_human` when:

- requirements conflict,
- dependencies are missing,
- a high-risk command is required,
- the harness behavior is internally inconsistent,
- evaluator/reviewer logic cannot safely determine an outcome,
- running forward would require changing policy rather than implementation.

## Final instruction

In this repository, correctness of the **workflow** is part of the product. Do not optimize only for feature output; optimize for traceable, evaluable, repeatable execution.

---
> Source: [human-again/orbit](https://github.com/human-again/orbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
