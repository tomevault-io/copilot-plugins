## overnight

> another you for when you're asleep — adaptive Claude Code message prediction.

# overnight

another you for when you're asleep — adaptive Claude Code message prediction.

## How it works

1. User works with Claude Code normally during the day — overnight reads session files passively
2. Before bed: `overnight` launches interactive TUI, suggests plans from recent activity
3. overnight adaptively predicts messages: predict one → execute via `claude -p` → observe output/diff/tests → predict next
4. All work happens on a single `overnight/{run-id}` branch with sequential commits
5. Morning: `overnight log` shows results — user reviews/merges the branch

## Architecture

```
src/
  cli.ts         — CLI entry point (start, stop, log, history, config)
  types.ts       — Core types and constants
  history.ts     — Extract messages from Claude Code session JSONL files
  predictor.ts   — Intent + history + results → adaptive message prediction (Anthropic API)
  executor.ts    — Adaptive execution loop, single branch per run
  interactive.ts — Interactive TUI orchestration (Anthropic SDK streaming + React Ink)
  profile.ts     — User profile built from conversation history
  ui/            — React Ink TUI components (theme, status, messages, input, etc.)
```

## Key decisions

- **Adaptive prediction** — not batch. Each message sees previous results (output, diff, test status) before predicting next
- **No proxy, no hooks** — reads existing `~/.claude/projects/` session files
- **Single branch per run** — all predictions commit to `overnight/{run-id}`, no per-message worktrees
- **Two run modes** — "Stick to plan" (one sprint, stop when goals met) and "Don't stop" (continuous sprints + tech debt)
- **Ambition levels** — safe/normal/yolo control prediction creativity
- **User-model prediction** — predictor never sees raw conversation history; instead gets user profile (WHO) + direction (WHERE) + workspace snapshot (WHAT EXISTS)
- **Claude Code CLI flags**: `-p` (non-interactive), `--permission-mode auto`

## Build

```bash
bun build src/cli.ts --outdir dist --target node
```

## Config

Stored in `~/.overnight/config.json`. Runs stored in `~/.overnight/runs/`.

---
> Source: [yail259/overnight](https://github.com/yail259/overnight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
