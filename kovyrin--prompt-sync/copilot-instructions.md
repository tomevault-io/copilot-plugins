## prompt-sync

> 1. **NEVER create commits automatically** - Only commit when explicitly requested by the user

# Git Operations Guidelines

## Commit Rules

1. **NEVER create commits automatically** - Only commit when explicitly requested by the user
2. **No implicit commits** - Even if changes seem complete or ready, do not commit without explicit instruction. You can ask for permission from the user and then commit if they agree.
3. **User phrases that mean "commit"** - Only commit when the user explicitly says things like:
   - "commit this"
   - "please commit"
   - "create a commit"
   - "git commit"
   - Similar explicit requests

## Push Rules

1. **NEVER push to remote repositories** - Under no circumstances should you run `git push`
2. **No automatic pushes** - Even if a commit is created, never push it
3. **User responsibility** - Pushing to remote is always the user's responsibility

## Allowed Git Operations

You MAY run these commands when helpful:
- `git status` - Check current state
- `git diff` - Show changes
- `git add` - Stage files (only when preparing for a user-requested commit)
- `git log` - Show commit history
- `git branch` - List or check branches
- `git restore` - Unstage or restore files

## Example Scenarios

❌ **Wrong**: "I've made all the changes and committed them for you"
✅ **Right**: "I've made all the changes. The files are ready if you'd like to commit them"

❌ **Wrong**: Making a commit after completing a task
✅ **Right**: Completing the task and informing the user the changes are ready

❌ **Wrong**: Running `git push` after creating a commit
✅ **Right**: Never running `git push` at all

## Summary

- Wait for explicit commit requests
- Never push to remote
- Focus on making changes, not managing version control
- Let the user control their git workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kovyrin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
