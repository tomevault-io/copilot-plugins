## workbench

> agents-workbench workflow — worktree-based development with branch detection

# agents-workbench Workflow

## Branch Detection
Before editing any source code, check: `git branch --show-current`
- If output is `agents-workbench` → source code is READ-ONLY. Suggest a worktree.
- If inside a `.worktrees/` path → you are in a worktree. Proceed normally.
- If on any other branch → normal development. Follow project conventions.

## Branch: agents-workbench (local-only, NEVER push)
- Coordination hub: AGENTS.md, .agents/*, docs/plans/*
- Source code is READ-ONLY on this branch
- Local commits only (preserve agent state)

## Implementation: ALWAYS in worktrees
- Create: `git worktree add .worktrees/<name> -b <branch> <default-branch>`
- Each feature/fix gets its own worktree from the default branch
- Push feature branches from worktrees, create PRs to default branch
- Clean up after merge: `git worktree remove .worktrees/<name>`

## What is editable on agents-workbench
- OK to edit: AGENTS.md, .agents/*, docs/plans/*, CLAUDE.md, .cursor/rules/*
- OK to read: any file (for context and planning)
- NEVER modify source code directly

## Agent Teams (Cursor + Claude Code)
- Lead agent: stays on agents-workbench, coordinates via AGENTS.md
- Worker agents: each in own worktree, parallel implementation
- All commits in worktrees use: `git commit -s -S`

## Setup
Run `~/.claude/scripts/setup-workbench.sh` in any project

---
> Source: [ArangoGutierrez/promptsLibrary](https://github.com/ArangoGutierrez/promptsLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
