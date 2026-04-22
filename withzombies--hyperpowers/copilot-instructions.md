## hyperpowers

> This repository ships workflow guidance for both Claude Code and Codex.

# AGENTS.md

This repository ships workflow guidance for both Claude Code and Codex.

## What This Repo Is

Hyperpowers is a workflow pack for AI coding agents:
- `skills/` contains Claude plugin skills
- `.agents/skills/` contains Codex-compatible skills
- `commands/` contains Claude slash-command shims
- `agents/` contains specialized subagent prompts
- `hooks/` contains Claude hook scripts and shared hook utilities

The goal is to improve the plugin itself. When a user talks about a failure from some other session or project, treat it as evidence for a missing workflow guardrail here. Do not try to debug the historical incident directly.

## Active Work Tracking

Active work uses local markdown task directories under `plans/active/`.

Task directories under `plans/active/<slug>/` are local working state:
- keep `plans/README.md`, templates, and placeholders tracked
- keep per-task directories out of git
- delete finished task directories instead of archiving them

Use:
- `plans/active/<slug>/plan.md` for the approved spec
- `plans/active/<slug>/context.md` for discoveries and resume notes
- `plans/active/<slug>/tasks.md` for the rolling backlog

Completed work:
- delete the local `plans/active/<slug>/` directory after verification

## Default Workflow

For substantial work:
1. Use `brainstorming` to shape the work and agree on goals, anti-goals, constraints, and approach.
2. Create a task directory in `plans/active/<slug>/`.
3. Research the feature or task with at least 3 internet sources focused on current best practices.
   Use a real web-search tool call or the `internet-researcher` agent. Memory does not count as research.
4. Put the approved spec in `plan.md`.
5. Use `writing-plans` to distill the spec into `context.md` and `tasks.md`.
6. Use `executing-plans` to work only the current `Now` slice.
7. Research new task slices with at least 3 sources before promoting them into active work.
   Use a real web-search tool call or the `internet-researcher` agent. Do not infer best practices from memory.
8. Update `context.md` and `tasks.md` immediately as reality changes.
9. Use `review-implementation` and `verification-before-completion` before claiming completion.

## Task Doc Rules

- `plan.md` is mostly stable after approval.
- `tasks.md` is intentionally fluid. Use `Now`, `Next`, `Later`, `Blocked`, and `Done`.
- Keep only 1-2 unchecked items in `Now`.
- Record rejected approaches and anti-goals in `plan.md` so implementation does not drift.
- Record resume notes in `context.md` before stopping or compacting.
- Delete the finished local task directory instead of moving it to an archive.

## Tool-Specific Notes

- Claude Code hooks should guide agents toward the markdown task docs.
- Codex should use the same task directories and repo docs, with `AGENTS.md` as the primary cross-tool policy file.
- Keep repo-specific process in repo docs; keep reusable operational guidance in skills.

---
> Source: [withzombies/hyperpowers](https://github.com/withzombies/hyperpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
