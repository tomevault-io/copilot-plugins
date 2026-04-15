## cursorskilltemplate

> General development guide with documentation and memory workflows


# Development Guide

## Project Structure

All code follows this layout:
- `src/dev/` - Source files (production code)
- `src/Tests/` - Test files (mirror structure from dev)

Use the `create-project` skill when scaffolding new features.

## Documentation Workflow

When a user requests code, libraries, or implementation help:

### Step 1: Check Memory First

Query the `user-memory` MCP for existing knowledge:

```
search_nodes → Look for relevant entities
open_nodes → Get details if found
```

### Step 2: Fetch Fresh Documentation

If memory lacks info or user needs current docs, use `user-context7`:

```
1. resolve-library-id → Get the library's Context7 ID
2. query-docs → Fetch current documentation
```

### Step 3: Store in Memory

After fetching useful documentation, persist it:

```
1. create_entities → Create entity for the library/topic
2. add_observations → Store key findings, patterns, examples
3. create_relations → Link related concepts
```

## Example Flow

**User**: "How do I use React Query for data fetching?"

1. **Check memory**: `search_nodes` for "react-query"
2. **If not found**: 
   - `resolve-library-id` → "@tanstack/react-query"
   - `query-docs` → Get latest patterns
3. **Store**: Create entity "react-query" with observations about usage patterns
4. **Respond**: Provide code using current best practices

## Available Tools

- See `.cursor/DEVELOPMENTTOOLS.md` for full MCP/tool reference
- Use `document-dev-tools` skill to update tool documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dezverev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dezverev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
