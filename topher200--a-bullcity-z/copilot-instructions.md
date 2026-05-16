## a-bullcity-z

> This project uses **bd (beads)** for issue tracking.

## Beads (bd) Issue Tracking

This project uses **bd (beads)** for issue tracking.

## Core Rules
- Track strategic work in beads (multi-session, dependencies, discovered work)
- Use `bd create` for issues, TodoWrite for simple single-session execution
- When in doubt, prefer bd persistence
- Session management: check `bd ready` for available work

## Common Workflows

### Finding Work
- `bd ready` - Show issues ready to work (no blockers)
- `bd list --status=open` - All open issues
- `bd list --status=in_progress` - Your active work
- `bd show <id>` - Detailed issue view with dependencies

### Creating & Updating
- `bd create --title="..." --type=task|bug|feature --priority=2` - New issue
  - Priority: 0-4 or P0-P4 (0=critical, 2=medium, 4=backlog). NOT "high"/"medium"/"low"
- **REQUIRED**: Use epics for any task requiring more than 1 bead (sub-task breakdown)
  - `bd create --title="..." --type=epic --priority=2` - Create epic
  - `bd create --title="..." --type=task --parent=<epic-id> --priority=2` - Create task under epic
  - **The --parent flag is REQUIRED** to ensure tasks get the correct prefix
- `bd update <id> --status=in_progress` - Claim work
- `bd update <id> --assignee=username` - Assign to someone
- `bd close <id>` - Mark complete
- `bd close <id1> <id2> ...` - Close multiple issues at once (more efficient)
- `bd close <id> --reason="explanation"` - Close with reason
- **Tip**: When creating multiple issues/tasks/epics, use parallel subagents for efficiency

### Dependencies & Blocking
- `bd dep add <issue> <depends-on>` - Add dependency (issue depends on depends-on)
- `bd blocked` - Show all blocked issues
- `bd show <id>` - See what's blocking/blocked by this issue

**Starting work:**
```bash
bd ready           # Find available work
bd show <id>       # Review issue details
bd update <id> --status=in_progress  # Claim it
```

**Completing work:**
```bash
bd close <id1> <id2> ...    # Close all completed issues at once
```

**Creating work with epics (REQUIRED for multi-bead tasks):**
```bash
# 1. Create epic for work requiring multiple sub-tasks
bd create --title="Implement feature X" --type=epic --priority=2

# 2. Create tasks under epic with --parent flag (REQUIRED for correct prefix)
# Run bd create commands in parallel (use subagents for many items)
bd create --title="Backend implementation" --type=task --parent=<epic-id> --priority=2
bd create --title="Frontend implementation" --type=task --parent=<epic-id> --priority=2
bd create --title="Write tests for X" --type=task --parent=<epic-id> --priority=2

# 3. Add dependencies between tasks if needed
bd dep add <test-task-id> <backend-task-id>  # Tests depend on backend
```

---
> Source: [topher200/a-bullcity-z](https://github.com/topher200/a-bullcity-z) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
