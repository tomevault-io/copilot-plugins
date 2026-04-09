## roacm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow

### Trunk-Based Development

- **Push directly to main** - no pull requests unless there's specific risk to mitigate
- **If risk exists**, present it to the user with reasoning and let them approve a PR
- Examples of risk: large refactors, breaking API changes, untested edge cases
- Default assumption: trunk-based is safe when tests pass

### User Approval Required

- **Never commit without checking with the user first** - explain what will be committed and why
- **Never push without asking the user** - always confirm before pushing to remote
- **If a change must temporarily break the system, stop and ask** - don't proceed with breaking changes without explicit approval

### Commit Granularity

- **Each commit should be the smallest independent change** that doesn't break the system
- **Prefer many small commits over few large ones** - each commit should do one thing
- **Close issues via commit message** when possible (e.g., `Fixes #38`) to link fixes to commits

### Commit Message Format

Follow [cbea.ms/git-commit](https://cbea.ms/git-commit/) guidelines:

1. Separate subject from body with a blank line
2. Keep subject line concise (~50 chars, avoid exceeding 72)
3. Capitalize the subject line
4. Do not end the subject line with a period
5. Use imperative mood ("Add feature" not "Added feature")
6. Use body to explain **what and why**, not how

### Squashing Local Commits

- **If a new change logically belongs with an unpushed commit**, recommend squashing to the user
- Use `git commit --fixup=<commit>` then `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <commit>~1`
- Only squash commits that haven't been pushed to remote
- This keeps history clean while maintaining small, focused commits during development

### Pre-commit and CI

- **Ensure pre-commit hooks are installed** before committing - verify hooks run successfully
- **After pushing, check pipeline status in the background** - monitor CI and alert user if it fails

## Task Management

- **Finding work**: Use `gh issue list --state open --limit 100` to fetch all open issues
- **After completing a card**: Check what's open. If only system cards (bugs, enhancements), prioritize bugs > quick wins > enhancements. If only content cards, pick highest priority. If both types are open, ask which to work on - default assumption is system improvements unless told otherwise.
- **Checking pipeline**: Use `gh run list --limit 20 --workflow=ci.yml`. Tests only run when theme files change; look for 3-5+ min runs (not 1-2 min build-only runs)

### Worktree Workflow

Use Git worktrees to work on multiple issues in parallel without switching branches or stashing:

**Commands:**

- `/pickup-on-worktree <issue-number>` - Create dedicated worktree for an issue
- `/pickup-on-worktree` - List all active worktrees with ports
- `./scripts/worktree-list.sh` - Detailed worktree status with merge info
- `./scripts/worktree-cleanup.sh` - Interactive cleanup of old worktrees

**Port Assignment:**

- Main repo uses port 4000
- Worktrees use ports 4001-4010 (10 parallel worktrees max)
- Port embedded in worktree name: `../roacm-4001-issue-123`
- When all ports in use, cleanup required before creating new worktree

**Naming Convention:**

- Worktree path: `../roacm-<PORT>-issue-<NUMBER>`
- Branch name: `issue-<NUMBER>`
- Location: Parallel to main repo, not inside it

**Context Tracking:**

- After creating worktree, Claude automatically switches working context to it
- All subsequent commands run from the worktree until context changes
- Switch contexts with: "Switch to main repo", "Switch to worktree 4002"
- Claude periodically confirms current working location

**Workflow:**

1. Create worktree: `/pickup-on-worktree 123`
2. Claude switches context automatically
3. Work on the issue (tests, commits, etc.)
4. Push changes: `git push -u origin issue-123`
5. Create PR from worktree
6. Switch to another context or clean up: `git worktree remove ../roacm-4001-issue-123`

**Benefits:**

- Work on multiple issues simultaneously
- No branch switching or stashing
- Separate Jekyll servers per worktree (different ports)
- Clean separation of work-in-progress

## Context Files for Different Work Types

**Load context based on the type of work:**

### Website/Theme Development

- **Read `docs/context/DEVELOPMENT.md`** - Commands, file structure, architecture
- **Read `docs/context/TESTING.md`** - Testing requirements and workflows
- **Read `docs/context/VISUAL_STYLE.md`** - Visual design, colors, accessibility (when making CSS/styling changes)

### Blog Content

- **Read `docs/context/TAGS.md`** - Canonical tag list
- **Read `docs/context/CATEGORIES.md`** - Valid categories
- **Read `docs/context/CONTENT.md`** - Content guidelines
- **Read `docs/context/STYLE.md`** - Writing style
- **Read `docs/context/CROSS-POSTING.md`** - Cross-posting to dev.to and Medium

**After publishing a post**, remind the user:

1. **dev.to**: Automatic if `devto: true` is set (verify in workflow run)
2. **Medium**: Manual import required - use Medium's "Import a story" feature

### Homepage/Landing Pages

- **Read `docs/context/HOMEPAGE.md`** - Homepage structure

**Do not load all context files at once - only load what's relevant to the current task.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/javatarz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/javatarz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
