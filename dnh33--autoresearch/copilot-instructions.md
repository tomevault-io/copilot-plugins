## autoresearch

> If `autoresearch.md` exists in the project root, **autoresearch mode is ACTIVE**.

# Autoresearch — Autonomous Experiment Loop

## Autoresearch Mode

If `autoresearch.md` exists in the project root, **autoresearch mode is ACTIVE**.

When autoresearch mode is active:
- Read `autoresearch.md` at the start of every session and after compaction — it contains the full experiment context.
- Use the helper scripts in `.claude/skills/autoresearch/scripts/` to run experiments:
  - `init-experiment.sh` — initialize/re-initialize session config
  - `run-experiment.sh` — run a timed experiment with output capture
  - `log-experiment.sh` — record results (keep/discard/crash/checks_failed)
  - `dashboard.sh` — open interactive HTML dashboard in browser
  - `summary.sh` — quick terminal summary of experiment progress
- **NEVER STOP looping** until the user interrupts. The user may be away for hours.
- After each experiment: edit code → commit idea → run benchmark → log result → keep or revert → repeat.
- Write promising but deferred optimizations to `autoresearch.ideas.md`.
- Update `autoresearch.md` "What's Been Tried" section periodically so resuming agents have full context.
- If the user sends a message during an experiment, finish the current run + log cycle first, then address their message.

## State Files

- `autoresearch.jsonl` — append-only log of every run (survives restarts and context resets)
- `autoresearch.md` — living session document (objective, metrics, scope, what's been tried)
- `autoresearch.sh` — benchmark script
- `autoresearch.checks.sh` — optional correctness checks (tests, types, lint)
- `autoresearch.ideas.md` — deferred optimization ideas backlog

## Starting Autoresearch

Use `/autoresearch` to start or resume an experiment loop.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dnh33) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
