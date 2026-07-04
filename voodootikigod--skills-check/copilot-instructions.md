## skills-check

> This project uses **GitHub Issues** for issue tracking. Use the GitHub CLI (`gh`) to manage issues and track your work.

# Agent Instructions

This project uses **GitHub Issues** for issue tracking. Use the GitHub CLI (`gh`) to manage issues and track your work.

## Quick Reference

```bash
gh issue list                  # List open issues for the repository
gh issue view <number>         # View details of a specific issue
gh issue edit <number> --assignee "@me" # Claim / assign an issue to yourself
gh issue close <number>        # Complete work and close an issue
```

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

## Issue Tracking with GitHub Issues

**IMPORTANT**: This project uses **GitHub Issues** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Quick Start

**Check for ready work:**

```bash
gh issue list --label "spec-alignment"
```

**Create new issues:**

```bash
gh issue create --title "Issue title" --body "Detailed context" --label "bug|feature|task" --label "priority: high|medium|low"
```

**Claim and update:**

```bash
gh issue edit <number> --assignee "@me"
```

**Complete work:**

```bash
gh issue close <number>
```

### Issue Labels & Types

We use GitHub labels to categorize work:
- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `spec-alignment` - Gaps with Agent Skills specification

### Priorities

- `priority: high` - Critical / high priority
- `priority: medium` - Medium priority
- `priority: low` - Low priority / backlog

### Workflow for AI Agents

1. **Check ready work**: List issues using `gh issue list`
2. **Claim your task**: Assign the issue to yourself using `gh issue edit <number> --assignee "@me"`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create a linked issue:
   - `gh issue create --title "Found bug" --body "Details... (discovered during work on #[parent-number])" --label "bug" --label "priority: high"`
5. **Complete**: Close the issue using `gh issue close <number>`

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Source: [voodootikigod/skills-check](https://github.com/voodootikigod/skills-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
