## blocked-git-commands

> Prevent destructive git commands that discard uncommitted work


# Blocked Git Commands

NEVER run the following git commands:

- `git checkout` — can silently discard uncommitted changes
- `git restore` — can silently discard uncommitted changes

If you need to switch branches, use `git switch` instead.
If you need to discard changes, ask the user for explicit confirmation first and explain what will be lost.

---
> Source: [Kevin-Liu-01/Princeton-Tower-Defense](https://github.com/Kevin-Liu-01/Princeton-Tower-Defense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
