## cursor

> Keep README.md in sync with command files in `commands/` directory and shared rules in `rules/` directory. Automatically update README when commands or shared rules change.

# README.md Sync Rule

## Purpose

Keep README.md in sync with command files in `commands/` directory and shared rules in `rules/` directory. Automatically update README when commands or shared rules change.

## How It Works

**When command files change:**
1. Extract description from the `## Overview` section
2. **Ask the user**: "Is this command Stable or WIP?" (if status changes)
3. Update the corresponding row in README.md table
4. Maintain table format: `| [**command&#8209;name**](commands/filename.md) | Status Badge | Description |`
5. Use non-breaking hyphens (`&#8209;`) in command names to prevent wrapping

**Operations:**
- **New command** → Add new table row (ask user for status: Stable or WIP)
- **Modified command** → Update description in table (ask user if status changed)
- **Renamed command** → Update link and name in table
- **Deleted command** → Remove table row
- **Status change** → Ask user and update badge

## Example

Command file `commands/new-command.md`:
```markdown
# New Command

## Overview
This command does something useful.
```

Becomes table row in README.md:
```markdown
| [**new&#8209;command**](commands/new-command.md) | ![WIP](https://img.shields.io/badge/status-WIP-yellow) | This command does something useful. |
```

**Status Badges:**
- Stable: `![Stable](https://img.shields.io/badge/status-stable-green)`
- WIP: `![WIP](https://img.shields.io/badge/status-WIP-yellow)`

## Automation

Update README.md automatically when command files change. No permission needed.

## Updating This Rule

When README.md structure is manually changed:
- Update this readme-sync.mdc file to reflect new structure
- Update section descriptions in "README.md Structure"
- Update examples to match new format
- Keep this rule synchronized with actual README.md organization
- This ensures future automated updates follow correct structure

## README.md Structure

The README.md maintains several sections:

### Quick Start Section

Section with usage instructions:
- **Location**: After introduction, before System Commands
- **Content**: Instructions for new and existing projects, shared rules overview
- **Maintenance**: Update when workflow changes
- **Avoid specific quantities**: Don't mention exact number of rules/commands

### Shared Rules Subsection (within Quick Start)

Lists shared rule topics within Quick Start:
- **Location**: Within Quick Start section, after For New/Existing Projects
- **Content**: List of shared rule topics (not filenames)
- **Maintenance**: Update when shared rules added, removed, or reorganized
- **When to update:**
  - Shared rule added → add topic to bullet list
  - Shared rule removed → remove topic from list
  - Rule topic reorganized → update description
  - Keep synchronized with actual rules in `/rules/` directory
- **Avoid specific quantities**: Don't say "10 rules" or exact count
- **List topics, not files**: Describe what rules cover, not `shared-*.mdc` filenames

**Example:**
```markdown
### Shared Rules

The shared rules provide universal guidance:
- Rules and commands management
- Version control and collaboration
- Development workflow (planning, features, bugs, refactoring)
- Code quality and standards
- Testing conventions
- Security practices
- Documentation standards
- Environment setup
- Release management

See [`rules/`](rules/) directory for all shared rules.
```

**When shared rules change:**
- New rule created: add topic to list
- Rule deleted: remove topic from list
- Rule renamed/reorganized: update topic description
- Keep list in sync with `/rules/` directory content

### Command Tables Section

Two separate tables for different command types:

**System Commands** - Workspace maintenance commands:
- Sync commands, workspace management
- Example: `sync-shared-rules`
- Format: `| [**command&#8209;name**](commands/filename.md) | Status Badge | Description |`

**Development Commands** - Project workflow commands:
- Auto-synced from `commands/` directory files
- Updated when command files change (see "How It Works" above)
- Format: `| [**command&#8209;name**](commands/filename.md) | Status Badge | Description |`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hacker-cb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
