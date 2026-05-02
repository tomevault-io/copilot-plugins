## git-safety

> NEVER commit, push, or create tags without explicit user confirmation


# Git Safety Rules

**NEVER run these git commands without asking the user first and getting explicit confirmation:**

- `git commit`
- `git push`
- `git tag`
- `git merge`
- `git rebase`
- `git reset`
- `git checkout -b` (creating branches)

## Required Workflow

1. **Show the user** what you're about to do (the exact commands)
2. **Ask explicitly**: "Would you like me to run these commands?"
3. **Wait for confirmation** before executing
4. Only proceed if the user says yes/confirm/go ahead

## Example

```
❌ BAD: Just running commands
git add . && git commit -m "fix" && git push

✅ GOOD: Ask first
"Here are the git commands to commit and push:
git add .
git commit -m "fix: description"
git push origin main

Would you like me to run these?"
```

This rule has no exceptions. Always ask, even if the user previously said "yes" to preparing changes.

---
> Source: [ChuckJonas/duckdb-vscode](https://github.com/ChuckJonas/duckdb-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
