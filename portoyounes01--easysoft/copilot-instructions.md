## progress-tracking

> Progress tracking and reflection requirements


# Progress Tracking Rule

**CRITICAL REQUIREMENT**: At EVERY STEP of any coding task, the agent MUST ABSOLUTELY:

1. **Reflect on changes**: Explicitly identify and analyze what has changed, what is changing, or what will change in the codebase
2. **Document progress**: Update appropriate task files to reflect the current state

## Task File Hierarchy:

### Main Tracking Files (Permanent):
- `TODO.md` - for pending tasks and future work
- `IN_PROGRESS.md` - for currently active tasks  
- `DONE.md` - for completed tasks and accomplishments

### Temporary Tracking Files (Session-based):
- `TEMP_TASKS_YYYY-MM-DD.md` - for daily subtasks, minor changes, debugging steps
- `SESSION_NOTES_[identifier].md` - for specific feature development sessions
- `TEMP_DEBUG_[issue].md` - for troubleshooting and bug investigation steps

## Usage Guidelines:

### Use Main Files For:
- Major features and milestones
- Important architectural decisions
- User-requested functionality
- Critical bug fixes
- Tasks that affect multiple components

### Use Temporary Files For:
- Minor UI adjustments and styling tweaks
- Debugging steps and investigation notes
- Experimental code or proof-of-concepts
- Refactoring individual functions
- Code cleanup and optimization
- Temporary workarounds

## Requirements:

- **NEVER skip this step** - even for minor changes or single-line edits
- **Be specific** about what changed (files, functions, features, etc.)
- **Update task status** to accurately reflect current progress
- **Cross-reference** completed work with original requirements
- **Note dependencies** between tasks when relevant
- **Promote important items** from temporary to main files when appropriate
- **Clean up temporary files** weekly or at session end
- **Archive valuable insights** from temporary files before cleanup

## Cleanup Process:

1. **Weekly Review**: Every 7 days, review all temporary files
2. **Promote Important Items**: Move significant accomplishments to main DONE.md
3. **Archive Insights**: Save valuable debugging insights or lessons learned
4. **Delete Obsolete Files**: Remove temporary files that are no longer relevant
5. **Consolidate Notes**: Merge related temporary tracking into main files when appropriate

This rule ensures complete transparency and accountability while maintaining organized, focused main tracking files.

---
> Source: [portoyounes01/easysoft](https://github.com/portoyounes01/easysoft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
