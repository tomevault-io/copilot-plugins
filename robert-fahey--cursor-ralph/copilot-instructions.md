## cursor-ralph

> This file contains project conventions and patterns discovered during development.

# Project Rules for Cursor

This file contains project conventions and patterns discovered during development.
Ralph (the autonomous agent loop) updates this file with learnings from each iteration.

## Project Structure

```
/
├── ralph.sh          # Main entry point
├── plan.sh           # Plan mode implementation
├── execute.sh        # Execution loop
├── prompt.md         # Cursor instructions for execution
├── plan-prompt.md    # Cursor instructions for analysis
├── prd.json          # Current PRD (user stories)
├── progress.txt      # Append-only learnings log
└── .cursorrules      # This file - project conventions
```

## Conventions

### Code Style

- Follow existing patterns in the codebase
- Use consistent naming conventions
- Add comments for complex logic

### Commits

- Format: `[STORY-ID] Brief description`
- One story per commit when possible
- Keep commits atomic and focused

### Quality Checks

- All code must pass type checking
- All code must pass linting
- All tests must pass
- Add tests for new functionality

## Patterns Discovered

<!-- Ralph will append patterns here during execution -->

## Gotchas

<!-- Ralph will append gotchas here during execution -->

## Useful Context

<!-- Ralph will append useful context here during execution -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robert-fahey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
