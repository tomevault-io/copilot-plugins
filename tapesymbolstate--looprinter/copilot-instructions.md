## looprinter

> looprinter is a loop template repository. `loop.sh` is a self-contained template for building any kind of iterative agent harness. Copy it, edit the prompt functions, run it.

# CLAUDE.md

## What This Project Is

looprinter is a loop template repository. `loop.sh` is a self-contained template for building any kind of iterative agent harness. Copy it, edit the prompt functions, run it.

## Architecture

```
loop.sh                              — the template (prompts + engine in one file)
working-records/                     — JSONL logs per run [gitignored]
output/                              — runtime artifacts (plan.json, progress.txt) [gitignored]
.claude/skills/looprinter-interview/ — interactive harness configuration skill
.claude/skills/looprinter-executor/  — loop execution and monitoring skill
```

## Core Concepts

### 1. Two Gates: Cycle Verify + Key Objective

The loop has two completion gates, and both must pass for the loop to exit:

- `verify()` — cycle-level. *Did the current cycle's plan execute correctly?* Schema, files, all tasks `passes:true`.
- `verify_objective()` — global. *Is the key objective truly met?* Pinned to `KEY_OBJECTIVE` (one-line north star at the top of `loop.sh`).

If cycle verify passes but `verify_objective()` returns 1, the cycle archives the plan as `working-records/plan_cycle_<N>_objective_gap.json` and re-plans the next increment with the gap injected into the replan prompt. The loop only stops when both gates align on the same cycle — that is the brand promise of "key objective."

`verify_objective()` ships with a `LOOPRINTER_OBJECTIVE_TODO` marker that makes it pass-through with a warning. Removing the marker (which `/looprinter-interview` does automatically) activates strict enforcement.

### 2. Headless Mode

The loop spawns agents in headless mode (`codex exec`, `claude -p`). Each iteration is a fresh agent with a clean context window. State lives in the filesystem, not in agent memory.

### 3. Working Records

Every iteration appends to a JSONL record file in `working-records/`. Records are the loop's persistent memory — they survive context resets and compound across iterations.

- Every agent iteration MUST append to the record file
- Never truncate or overwrite records mid-run

### 4. Cronjob / Background Execution

The intended workflow: a main Claude Code session launches `loop.sh` as a background task or cronjob, then observes `working-records/` and stdout to improve the harness.

```
Main Claude Code session
  ├── launches loop.sh as cronjob/task (inner loop)
  ├── reads working-records/ to detect failure patterns + objective gap stalls
  └── edits loop.sh prompts / verify_objective() (outer loop)
```

The inner loop does the work. The outer loop (main agent) improves how the work gets done.

## Building a New Harness

Use `/looprinter-interview` to interactively configure a harness — it interviews you about the **key objective** and cycle deliverables, then writes `KEY_OBJECTIVE`, `verify_objective()`, the prompt functions, and the per-cycle `verify()` gate directly into `loop.sh`.

Or manually copy `loop.sh` and edit:

- `KEY_OBJECTIVE` — one-line north star, injected into all prompts
- `verify_objective()` — global completion gate (remove `LOOPRINTER_OBJECTIVE_TODO` to activate)
- `gen_plan_prompt()` — planning phase prompt
- `gen_build_prompt()` — build phase prompt
- `gen_replan_prompt()` — recovery prompt that handles cycle errors and/or objective gap (3 args)
- `verify()` — cycle-level quality gate (exit 0 = pass); default checks plan.json tasks and progress.txt
- `setup()` — one-time preprocessing
- `POST_PHASES` + `gen_<name>_prompt()` — optional phases after both gates pass

## Running the Loop

Use `/looprinter-executor` to launch, monitor, and manage the loop. Never run `loop.sh` in the foreground — it blocks the main session.

## Rules

- Keep prompt functions focused — one responsibility per phase
- `verify()` and `verify_objective()` must be fast and deterministic (no LLM calls)
- Cycle-level checks live in `verify()`; global "are we done?" checks live in `verify_objective()` — do not mix
- Prompts reference file paths the agent can read, not inline data
- The Key Objective block is auto-prepended to every prompt by `_objective_block` — do not duplicate it inside heredocs

---
> Source: [tapesymbolstate/looprinter](https://github.com/tapesymbolstate/looprinter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
