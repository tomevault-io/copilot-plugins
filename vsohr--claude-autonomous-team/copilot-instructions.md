## claude-autonomous-team

> - **Always use a git worktree** for any code changes. Never work directly on the main working tree.

# CLAUDE.md

## Git Worktrees

- **Always use a git worktree** for any code changes. Never work directly on the main working tree.
- Create worktrees inside this repo under `worktrees/<branch-name>`: `git worktree add worktrees/<branch-name> -b <branch-name>`
- Worktrees MUST live within the parent repo folder under `worktrees/` (and should be gitignored). Do not create sibling worktree directories outside the repo.
- This prevents clashes with running production containers and concurrent Claude Code sessions.

---
> Source: [vsohr/claude-autonomous-team](https://github.com/vsohr/claude-autonomous-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
