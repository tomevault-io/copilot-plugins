## thinking-memory-hub

> `Thinking Memory Hub` is a Python MCP-backed memory server for notes and tags.

# AGENTS

## Project

`Thinking Memory Hub` is a Python MCP-backed memory server for notes and tags.

Current implementation scope is the note-storage V1:

- `notes.save`
- `notes.search`
- `notes.update`
- `tags.search`

## Source Of Truth

Treat `docs/specs/` as the normative source of truth for implementation.

Read documents in this order when you need project context:

1. `docs/index.md`
2. `docs/specs/mcp_server.md`
3. `docs/specs/data_contracts.md`
4. `docs/specs/mcp_tool_definitions.md`
5. `docs/specs/storage_client_interface.md`

When documents conflict:

- `docs/specs/*` wins over any other project notes

## Architecture Rules

The key boundary is:

```text
app/backend code -> MCP memory server -> Postgres
```

Implementation constraints:

- MCP tool handlers validate input, call exactly one storage method, and serialize output
- SQL and transactions belong in `app/memory_server/storage_client.py`
- data models and enums belong in `app/memory_server/models.py`
- configuration loading belongs in `app/memory_server/config.py`

## Working Notes

Useful implementation entrypoints:

- `app/memory_server/main.py`
- `app/memory_server/server.py`
- `app/memory_server/mcp_tools.py`
- `app/memory_server/storage_client.py`
- `tests/`

## Testing

- unit tests should run without a live database
- integration tests are opt-in under `tests/integration/`
- integration tests use `TEST_DATABASE_URL`

---
> Source: [olesyastorchakprojects/thinking_memory_hub](https://github.com/olesyastorchakprojects/thinking_memory_hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
