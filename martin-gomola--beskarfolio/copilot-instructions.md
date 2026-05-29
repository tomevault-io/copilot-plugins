## beskarfolio

> **IMPORTANT**: When creating documentation markdown files (`.md`), place them in the correct location:

# BeskarFolio - File Organization Rules

## Documentation Files

**IMPORTANT**: When creating documentation markdown files (`.md`), place them in the correct location:

### Rules:

1. **All project documentation** → Place in `/docs/` directory
   - Examples: `docs/DEPLOYMENT_GUIDE.md`, `docs/API_DOCUMENTATION.md`, `docs/ARCHITECTURE.md`
   - Use descriptive UPPERCASE names with underscores

2. **Exceptions - Root level only**:
   - `README.md` (main project readme)
   - `CHANGELOG.md` (version history)
   - `CONTRIBUTING.md` (contribution guidelines)
   - `LICENSE.md` (project license)

3. **Never create documentation at project root** unless it's one of the exceptions above

### Examples:

✓ **Correct**:
```
docs/DEPLOYMENT_GUIDE.md
docs/PERFECTIONISM_LESSONS.md
docs/API_ENDPOINTS.md
README.md (root level - exception)
```

✗ **Incorrect**:
```
DEPLOYMENT_GUIDE.md (should be docs/DEPLOYMENT_GUIDE.md)
LESSONS_LEARNED.md (should be docs/LESSONS_LEARNED.md)
NOTES.md (should be docs/NOTES.md)
```

## Before Creating Documentation

1. Check if similar documentation already exists
2. Place in `/docs/` directory
3. Use descriptive, UPPERCASE names
4. Update main `README.md` if the new doc is important

<!-- code-review-graph MCP tools -->
## MCP Tools: code-review-graph

**IMPORTANT: This project has a knowledge graph. ALWAYS use the
code-review-graph MCP tools BEFORE using Grep/Glob/Read to explore
the codebase.** The graph is faster, cheaper (fewer tokens), and gives
you structural context (callers, dependents, test coverage) that file
scanning cannot.

### When to use graph tools FIRST

- **Exploring code**: `semantic_search_nodes` or `query_graph` instead of Grep
- **Understanding impact**: `get_impact_radius` instead of manually tracing imports
- **Code review**: `detect_changes` + `get_review_context` instead of reading entire files
- **Finding relationships**: `query_graph` with callers_of/callees_of/imports_of/tests_for
- **Architecture questions**: `get_architecture_overview` + `list_communities`

Fall back to Grep/Glob/Read **only** when the graph doesn't cover what you need.

### Key Tools

| Tool | Use when |
|------|----------|
| `detect_changes` | Reviewing code changes — gives risk-scored analysis |
| `get_review_context` | Need source snippets for review — token-efficient |
| `get_impact_radius` | Understanding blast radius of a change |
| `get_affected_flows` | Finding which execution paths are impacted |
| `query_graph` | Tracing callers, callees, imports, tests, dependencies |
| `semantic_search_nodes` | Finding functions/classes by name or keyword |
| `get_architecture_overview` | Understanding high-level codebase structure |
| `refactor_tool` | Planning renames, finding dead code |

### Workflow

1. The graph auto-updates on file changes (via hooks).
2. Use `detect_changes` for code review.
3. Use `get_affected_flows` to understand impact.
4. Use `query_graph` pattern="tests_for" to check coverage.

---
> Source: [martin-gomola/beskarfolio](https://github.com/martin-gomola/beskarfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
