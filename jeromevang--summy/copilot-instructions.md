## summy

> Summy is a development tool with:

# Summy Project Rules

## Project Overview
Summy is a development tool with:
- React/TypeScript frontend on port 5173
- Express API server on port 3001  
- RAG server for semantic code search on port 3002
- MCP server for tool integrations

## Memory System (File-Based)

### WORKING_MEMORY.md (Project Context)
Location: `./WORKING_MEMORY.md` (project root)
Purpose: Project-specific context that persists between sessions

**Always check this file first** when starting work on this project.

### GLOBAL_MEMORY.md (User Context)  
Location: `C:/Users/Jerome/GLOBAL_MEMORY.md`
Purpose: Cross-project info (preferences, personal context, tools setup)

**Check this for user preferences** that apply across all projects.

## When to Update Memory Files

Update WORKING_MEMORY.md when:
- A task completes
- Conversation exceeds 5-10 turns  
- Before switching tasks
- At natural stopping points

Update GLOBAL_MEMORY.md when:
- User preferences change
- New cross-project tools/setup documented
- Personal info that applies everywhere

## Memory File Format

```markdown
# WORKING_MEMORY (or GLOBAL_MEMORY)

## Current Goal
[Active focus]

## Completed  
- [Done items]

## Key Decisions
- [Decisions and rationale]

## Open Questions
- [Unresolved items]

## Next Actions
- [Immediate next steps]
```

## Context Management Rules ⚠️

1. **On session start:** Read WORKING_MEMORY.md and GLOBAL_MEMORY.md
2. **On "save context":** Update WORKING_MEMORY.md with current state
3. **On "load context":** Read both memory files and summarize
4. **When context gets long:** Summarize to WORKING_MEMORY.md proactively

## Coding Standards
- Use TypeScript for all new code
- Prefer functional components with hooks in React
- Use async/await over promises
- Error handling should be explicit

## Important Paths
- `server/src/` - Express API server
- `client/src/` - React frontend
- `rag-server/src/` - RAG/embeddings server
- `mcp-server/src/` - MCP tools server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeromevang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
