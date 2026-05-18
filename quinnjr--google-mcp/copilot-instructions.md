## auto-commit

> Commit changes after every completed agent task


# Auto-Commit After Agent Tasks

After completing any task that modifies files in the repository, the agent should commit the changes.

## Rules

1. **Commit after every completed task** - When a task is finished and files have been modified, stage and commit all changes before responding to the user.

2. **Use conventional commits** - Follow the git-flow commit message format:
   ```
   <type>(<scope>): <subject>
   ```

3. **Push if remote is configured** - After committing, push to the remote repository if one is set up.

4. **Atomic commits** - Each task should result in a single, focused commit that represents the complete change.

5. **Don't commit partial work** - Only commit when the task is fully complete and the code builds/lints successfully.

## Workflow

After completing a task:

```bash
git add -A
git commit -m "<type>(<scope>): <descriptive message>"
git push  # if remote exists
```

## Exceptions

Do NOT commit when:
- The user explicitly asks not to commit
- The task is exploratory (reading files, searching code)
- The changes don't compile or have lint errors
- The user is still iterating on the changes

---
> Source: [quinnjr/google-mcp](https://github.com/quinnjr/google-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
