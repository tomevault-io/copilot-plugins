## reincheck

> This is **reincheck**, a Python CLI tool for managing AI coding agents. Key files:

## Project Overview

This is **reincheck**, a Python CLI tool for managing AI coding agents. Key files:
- **reincheck/__init__.py**: Main logic - handles version checks, fetch_release_notes, commands from agents.yaml
- **reincheck/commands.py**: Click CLI definitions (cli, check, update, upgrade, install, list, release-notes)
- **reincheck/agents.yaml**: Configuration for 16 agents (crush, kilocode, opencode, claude, grok, gemini, cline, continue, interpreter, droid, openhands, mistral, codex, goose, roo, aider, kimi, amp)

Agent versions fetched via `get_latest_version()` (executes check_latest_command) and `fetch_release_notes()` (GitHub API for agents with github_repo). NPM info via `get_npm_release_info()`, PyPI via `get_pypi_release_info()`. 

## Tooling:

### Tools & libraries

Use `uv` to run script and manage Python packages. For further tooling `mise` is your friend.
CLI parts are written the the help of the Click library.
For non-destructive exploration: you have podman rights. So `podman run docker.io/jdxcode/mise` oder `podman run docker.io/homebrew/brew` might make things easier or possible

### Git Workflow & Commit Messages

- **Conventional Commits**: All commit messages **must** adhere to the Conventional Commits specification. This is enforced by a `commit-msg` git hook.
  - **Secrets**: NEVER commit any secrets unless it's an obivious placeholder (gho_xxxyyzzz123456) or you have been explicitly instructed by the user
  - **Format**: `<type>[optional scope]: <description>`
  - **Example**: `feat(converter): add vulkan backend support`
  - **Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`
  - **Enforcement**: The `pre-commit` framework with `conventional-pre-commit` hook is installed. If a commit message does not conform, the commit will be aborted.

### File Editing Strategy

- **Use the Right Tool for the Job**: For any non-trivial file modifications, you **must** use the advanced editing tools provided by the MCP server (e.g., `morph`).
  - **Simple Edits**: Use `replace` only for simple, unambiguous, single-line changes.
  - **Complex Edits**: For multi-line changes, refactoring, or context-aware modifications, try `fastapplyproxy` first.
  - 
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

If you are not aware of the 'SESSION CLOSE PROTOCOL' at this point run `bd prime` to get the full beads instructions
`PRIME.md` is the canonical checked-in snapshot of that protocol.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Auto-syncs to JSONL for version control
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update bd-42 --status in_progress --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

**IMPORTANT**: Have you run the CODE REVIEW & REFINEMENT PROTOCOL? It determines if you're allowed to close or if you need to halt here.

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task**: `bd update <id> --status in_progress`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
   - `bd create "Found bug" --description="Details about what was found" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Auto-Sync

bd automatically syncs with git:

- Exports to `.beads/issues.jsonl` after changes (5s debounce)
- Imports from JSONL when newer (e.g., after `git pull`)
- No manual export/import needed!

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems

For more details, see README.md and docs/QUICKSTART.md.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **BUT WAIT:** Now is the time to engage the CODE REVIEW & REFINEMENT PROTOCOL to see if you can close up shop or have extra steps
4. **Update issue status** - Close finished work, update in-progress items
5. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
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
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Megatherium)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Megatherium)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
