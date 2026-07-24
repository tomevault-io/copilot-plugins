## agentkits-memory

> Persistent memory system is active. Your decisions, patterns, errors, and context are saved across sessions.

# AgentKits Memory

Persistent memory system is active. Your decisions, patterns, errors, and context are saved across sessions.

## Available MCP Tools

| Tool | Purpose |
|------|---------|
| `memory_save` | Save decisions, patterns, errors, context |
| `memory_search` | Search index (Step 1/3 — lightweight) |
| `memory_timeline` | Get context around a result (Step 2/3) |
| `memory_details` | Fetch full content (Step 3/3) |
| `memory_recall` | Quick topic summary |
| `memory_list` | List recent memories |
| `memory_update` | Update existing memory |
| `memory_delete` | Delete memories |
| `memory_status` | Database health check |

## Workflow

1. `memory_status()` — Check if memories exist before searching
2. `memory_search(query)` — Get index with IDs (~50 tokens/result)
3. `memory_timeline(anchor="ID")` — Get context around interesting results
4. `memory_details(ids=["ID1","ID2"])` — Fetch full content ONLY for filtered IDs

Save important context naturally as you work — decisions made, patterns discovered, errors solved.

---
> Source: [aitytech/agentkits-memory](https://github.com/aitytech/agentkits-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
