## rooted-light

> This block is managed by `/devx-init`. Hand-edits inside the markers will

<!-- devx:start -->
# CLAUDE.md — Agent context for this project

This block is managed by `/devx-init`. Hand-edits inside the markers will
trigger an INTERVIEW.md merge-conflict entry on the next `/devx-init` run;
add your own context **outside** the markers (above or below).

## What this project is

Scaffolded non-interactively — describe what this project builds (see INTERVIEW.md).

## Strategic axes

| Axis | Setting |
|---|---|
| `mode` | **YOLO** |
| `project.shape` | **empty-dream** |
| `thoroughness` | **send-it** |

Source of truth: `devx.config.yaml`. Don't add one-off mode-aware logic
without updating `docs/MODES.md` first.

## Backlog files

| File | Written by | Read by |
|---|---|---|
| `DEV.md` | `/devx-plan`, `/devx`, ManageAgent | `/devx`, `/devx-test` |
| `PLAN.md` | `/devx-plan`, ManageAgent | `/devx-plan`, you |
| `TEST.md` | `/devx`, `/devx-test`, FocusAgent | `/devx-test` |
| `DEBUG.md` | any agent (CI red, flake, regression) | `/devx-debug` |
| `FOCUS.md` | FocusAgent, you | `/devx-plan`, `/devx-debug` |
| `INTERVIEW.md` | any agent blocked on a human decision | you |
| `MANUAL.md` | any agent when action requires a human | you |
| `LESSONS.md` | LearnAgent, you (`--add`) | ManageAgent, you, mobile |

INTERVIEW = decisions you must make. MANUAL = actions you must take. Don't
conflate.

## Spec file convention

```
<type>/<type>-<hash>-<timestamp>-<slug>.md
```

Frontmatter carries `hash`, `type`, `created`, `title`, `from:`, `spawned:`,
`status`, `owner`, `branch`. Body has Goal, Acceptance criteria, Technical
notes, Status log (append-only). The Status log is the request history —
where a thing came from, where it went.

## Working agreements

- **Don't duplicate business logic.** Wrap existing endpoints, tools,
  utilities.
- **One commit per story / sub-task.** Atomic, reviewable.
- **Fix forward.** If review finds issues, fix them in the same item; don't
  open follow-ups for in-scope work.
- **Status log is append-only.** Add lines; don't rewrite history.
- **Worktrees are isolation, not staging.** Don't run a non-`/devx` flow
  inside a worktree; don't share a worktree across agents.
<!-- devx:end -->

---
> Source: [LeoTheMighty/rooted-light](https://github.com/LeoTheMighty/rooted-light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
