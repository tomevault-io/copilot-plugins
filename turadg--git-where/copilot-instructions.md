## git-where

> - **All side-effects require user opt-in.** Read-only operations (looking up a path, listing repos) run unconditionally. Operations that modify the filesystem or git state (creating worktrees, fetching branches) require an explicit flag (e.g. `--create`). Never surprise the user with mutations.

# Design Principles

- **All side-effects require user opt-in.** Read-only operations (looking up a path, listing repos) run unconditionally. Operations that modify the filesystem or git state (creating worktrees, fetching branches) require an explicit flag (e.g. `--create`). Never surprise the user with mutations.

---
> Source: [turadg/git-where](https://github.com/turadg/git-where) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
