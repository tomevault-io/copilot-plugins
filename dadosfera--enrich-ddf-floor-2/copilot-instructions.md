## enrich-ddf-floor-2

> The `git sync` command is a custom workflow used in this project to synchronize local changes with the remote repository [[memory:2873774]] [[memory:2777283]].

# Git Sync Workflow

## Purpose
The `git sync` command is a custom workflow used in this project to synchronize local changes with the remote repository [[memory:2873774]] [[memory:2777283]].

## Workflow Steps
The git sync workflow performs the following operations:

1. **Commit all staged changes** (with pre-commit hooks)
   - All pre-commit hooks must pass (no --no-verify)
   - Fix any linting, formatting, or test errors before committing

2. **Pull latest changes from origin**
   - Use `git pull origin <branch> --rebase` to avoid merge commits
   - Resolve any conflicts if they occur

3. **Push changes to remote**
   - Use `git push origin <branch>`
   - Pre-push hooks will run automatically

## Important Notes
- **Never use `--no-verify`** flag [[memory:3060426]] [[memory:3060419]]
- **Never use `git reset`** to prevent destructive history changes [[memory:2777283]]
- **Always use timeout** when running git commands [[memory:2873774]]
- Environment variable `EMERGENCY_HEALTH_CHECK_SKIP=1` can be used to skip health checks when services aren't running

## Example Usage
```bash
# Stage changes
timeout 10 git add -A

# Commit with all pre-commit checks
timeout 60 git commit -m "feat: Your commit message"

# Pull latest changes
timeout 30 git pull origin main --rebase

# Push to remote
timeout 30 git push origin main
```

## Branch-Specific Workflow
When working on feature branches:
1. Create branch: `git checkout -b feature/branch-name`
2. Make changes and commit
3. Push branch: `git push origin feature/branch-name`
4. Switch back to main: `git checkout main`
5. Perform full git sync on main branch
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dadosfera) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
