## subetha

> - One active agent session must use one dedicated Git worktree and branch.

# SubEtha Agent Collaboration Rules

## 1. Worktree isolation is mandatory

- One active agent session must use one dedicated Git worktree and branch.
- When multiple sessions are active, never edit the main worktree directly.
- Never edit another session's worktree.
- Before starting work, verify the location with:
  - `git status --short --branch`
  - `git worktree list`
- If the current directory is not the assigned worktree, stop and report the problem.
- Do not overwrite or reset files created by another session.

Recommended naming:

```text
worktree: ../.worktrees/subetha/<session-id>
branch:   subetha/<session-id>
```

## 2. Workflow for non-trivial tasks

Follow the four phases in `CLAUDE.md`:

1. Requirements definition
2. Design
3. Task breakdown
4. Execution

Do not start implementation before the required preceding phase is complete and reviewed.

## 3. Agent responsibilities

- Claude Code: requirements, design, task breakdown, and implementation.
- Codex: independent review and verification.
- Codex review must not modify implementation or workflow files unless explicitly requested.
- Do not run two implementation agents against the same worktree or the same workflow files.

## 4. Workflow files

- Workflow files are worktree-local. Do not use a shared absolute path.
- Use `.claude_workflow/<session-id>/` for session-specific documents when parallel work is required.
- A session must not edit another session's requirements, design, or task files.
- Use `.claude_workflow/merged/` only for the designated integration session.
- Before editing any workflow file, read its current contents and preserve unrelated decisions.
- Never delete a file's contents merely because a new task has started.

## 5. Change policy

- Prefer additive changes.
- Do not remove or alter existing commands or behavior without explicit approval.
- Work incrementally and run the relevant tests after meaningful changes.
- If a conflict, unexpected change, or missing prerequisite is detected, stop and report it instead of overwriting files.

## 6. Repository-specific safety

- Do not vendor, copy, or redistribute the official `@zerc20/sdk` source.
- Respect the BUSL-1.1 boundary described in `CLAUDE.md`.
- Keep secrets and credentials out of tracked files.

---
> Source: [peaceandwhisky/SubEtha](https://github.com/peaceandwhisky/SubEtha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
