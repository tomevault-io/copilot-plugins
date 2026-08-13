## yes-my-lord

> AI coding assistants are powerful — but only when properly configured.

# Yes My Lord (YML) — Agent Instructions

## Project Overview

AI coding assistants are powerful — but only when properly configured. 
Setting up agents, skills, rules, hooks, and MCP servers from scratch 
for every new repository is repetitive, inconsistent, and error-prone.

This project is an open-source scaffolding CLI that generates 
production-ready configurations for Claude Code, Cursor, VS Code + Copilot, 
and OpenCode. One command bootstraps any repository with the right 
structure — agent rules, skill templates, hooks, commands, and MCP configs 
— tailored to your chosen IDE and workflow.

## What it scaffolds
- Agent rules & memory files (CLAUDE.md, .cursorrules, .github/copilot-instructions.md)
- Reusable skill & hook templates
- MCP server configurations
- Task management structure (todo.md, lessons.md)
- IDE-specific settings and command palettes

## Why Bun
Fast, zero-config, and native to the JS/TS ecosystem these tools live in.
No install required: `bunx scaffold-agent my-project`

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately – don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes – don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests – then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimat Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---
> Source: [openbutler/yes-my-lord](https://github.com/openbutler/yes-my-lord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
