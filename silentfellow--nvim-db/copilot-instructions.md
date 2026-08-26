## nvim-db

> `nvim-db` is a Neovim database client plugin written in Lua.

# AGENTS.md

## Project

`nvim-db` is a Neovim database client plugin written in Lua.

The goal is to provide a simple, native Neovim workflow for:

- Managing database connections.
- Storing query buffers per connection.
- Executing database queries.
- Displaying query results inside Neovim.

## Database Support

Current database support:

- PostgreSQL
- Redis
- SQLite
- MySQL
- MongoDB
- MariaDB
- DuckDB

Planned:

- Elasticsearch
- Cassandra
- ClickHouse
- DynamoDB
- CockroachDB

## Stack / Tools

- Lua / Neovim Lua API
- PostgreSQL (`psql`)
- Redis (`redis-cli`)
- StyLua — formatting
- LuaLS — Lua language server
- mise — development tool management

## Structure

```text
plugin/
└── nvim-db.lua       # User commands

lua/nvim-db/
├── init.lua          # Public API
├── ui.lua            # Main UI
├── connections.lua   # Connection management
├── buffers.lua       # Query buffer management
├── query.lua         # Query execution
├── config.lua        # config
└── utils.lua         # Utilities
```

## Development Guidelines

- Keep the implementation simple and readable.
- Prefer Neovim APIs over unnecessary abstractions.
- Keep UI, persistence, and database execution logic separated.
- Avoid adding dependencies unless necessary.
- Avoid premature abstractions such as an adapter framework.
- Follow the existing code style.
- Make focused changes; avoid unrelated refactors.
- Do not add configuration options unless there is a real use case.

## Formatting / Checks

Format with:

```bash
stylua .
```

Check formatting with:

```bash
stylua --check .
```

Use LuaLS for editor/type analysis.

## Important

This is an actively evolving project. Before making architectural changes, inspect the existing implementation and preserve the current design unless the requested feature requires otherwise.

---
> Source: [silentFellow/nvim-db](https://github.com/silentFellow/nvim-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
