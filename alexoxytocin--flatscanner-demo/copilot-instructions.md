## flatscanner-demo

> This repository uses `github/spec-kit` with two memory layers:

# AGENTS.md

This repository uses `github/spec-kit` with two memory layers:

- `docs/` for durable product and architecture context
- `specs/<feature-id>/` for feature intent, plan, and execution state

## Read Order

1. `.specify/memory/constitution.md`
2. `docs/README.md`
3. `docs/project-idea.md`
4. `docs/project/frontend/frontend-docs.md`
5. `docs/project/backend/backend-docs.md`
6. `docs/adr/*.md`
7. `specs/*/spec.md`
8. `specs/*/plan.md`
9. `specs/*/tasks.md`
10. Relevant implementation files

## Roles

- Codex owns architecture, review quality, CI/CD health, and local Claude orchestration.
- Claude Code is the primary implementation agent for product code.

## Boundaries

- Codex may directly edit docs, ADRs, specs, prompts, workflows, templates, and other process files.
- Product code in `src/`, `tests/`, and runtime setup must land through Claude-authored pull requests by default.
- Codex must not edit product code directly in the main checkout, even for small fixes, foundation slices, or emergency follow-ups.
- Multi-agent work must use isolated git worktrees; never run multiple coding agents in one working tree.

## Working Rules

- Start every task from current `main`.
- Keep changes scoped and avoid unrelated refactors.
- Update docs or specs before code when behavior or architecture changes.
- Keep one branch and one PR per Claude worker task.
- Before any product-code task, create or update the active `specs/<feature-id>/` folder first.
- Before any product-code task, explicitly use the standard loop: feature memory -> isolated Claude worktree -> PR -> required checks -> AI review.
- If the Claude worker, isolated worktree, or PR loop is unavailable, stop and report the blocker instead of implementing locally.
- Local draft product-code edits in `src/`, `tests/`, or runtime setup do not count as progress and must not be presented as task completion.
- Treat `docs/`, `specs/`, and `.specify/` as repository memory, not session memory.
- Keep `.github/workflows/` green.
- Automated PR review is selected only through the repo variable `AI_REVIEW_AGENT`.

## Completion

At task completion:

1. Update `specs/<feature-id>/tasks.md`.
2. Record durable decisions in `docs/` or `docs/adr/` when needed.
3. Capture follow-up work in the same feature folder or a new spec.

An orchestrated PR loop is finished only when the current PR head SHA has:

- no blocking review findings
- green required checks
- no merge conflicts
- only human approval or final merge remaining

If any of those are false, the task is still in progress unless the user explicitly pauses it.

Local-only product-code edits without an active PR loop are still in progress.

---
> Source: [AlexOxytocin/flatscanner-demo](https://github.com/AlexOxytocin/flatscanner-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
