## mcp-servers

> MCP server usage guidelines: Context7, MCP Debugger, PostgreSQL; roam CLI for code navigation


# MCP Server Rules

## Overview

This project uses multiple MCP servers, each optimized for specific tasks:

- **Context7**: External library documentation
- **MCP Debugger**: Interactive debugging (JavaScript/Python)
- **PostgreSQL** (`@modelcontextprotocol/server-postgres`): SQL against the app DB; URL from `.env` via `scripts/postgres-mcp-wrapper.sh` (`MCP_POSTGRES_URL` or `DATABASE_URL`)

**Code navigation** uses the **`roam`** CLI from devenv (not an MCP server). See **`.cursor/rules/roam.mdc`**.

## Agent Rules

### Library Documentation Agent Rules - Context7 MCP Server

**Context7 provides up-to-date documentation for libraries. Use these tools for all external library work:**

- **`resolve-library-id`**: ALWAYS use FIRST when you need documentation. Resolves package names to Context7 library IDs.
- **`query-docs`**: ALWAYS use AFTER resolving library ID to get documentation. Provides up-to-date API references and examples.

**Usage Pattern**: 
1. Use `resolve-library-id` to find the library ID
2. Use `query-docs` with the library ID to fetch documentation
3. NEVER guess API usage - ALWAYS query Context7 first

### Debugging Agent Rules - MCP Debugger MCP Server

**MCP Debugger provides interactive debugging capabilities. Use these tools for debugging JavaScript and Python:**

#### Session Management
- **`create_debug_session`**: ALWAYS use FIRST to create a debugging session for JavaScript or Python code.
- **`list_debug_sessions`**: Use to see all active debugging sessions.
- **`close_debug_session`**: Use to close a debugging session when done.
- **`list_supported_languages`**: Use to check which languages are supported for debugging.

#### Debugging Setup
- **`set_breakpoint`**: ALWAYS use to set breakpoints BEFORE starting debugging. Set on executable lines (assignments, function calls, conditionals, returns).
- **`start_debugging`**: ALWAYS use to launch debugging with proper script path and configuration.

#### Execution Control
- **`step_over`**: Use to execute current line and move to next line (step over function calls).
- **`step_into`**: Use to step into function calls to debug function internals.
- **`step_out`**: Use to step out of current function to caller.
- **`continue_execution`**: Use to continue execution until next breakpoint or end.
- **`pause_execution`**: Use to pause execution (Note: May not be fully implemented).

#### State Inspection
- **`get_local_variables`**: ALWAYS use to see local variables in current stack frame (most convenient).
- **`get_variables`**: Use to get variables for a specific scope (uses variablesReference from stack frame).
- **`get_stack_trace`**: ALWAYS use to see the call stack and current execution position.
- **`get_scopes`**: Use to get all scopes (local, closure, global) for a stack frame.

#### Advanced Debugging
- **`evaluate_expression`**: ALWAYS use to test hypotheses, inspect values, or modify program state during debugging.
- **`get_source_context`**: Use to view source code around a specific line with context lines.

**Usage Pattern**: 
1. `create_debug_session` with language
2. `set_breakpoint` on executable lines
3. `start_debugging` with script path
4. Use `step_over/into/out` and `continue_execution` to navigate
5. Use `get_local_variables`, `get_stack_trace`, `evaluate_expression` to inspect state
6. `close_debug_session` when done

### PostgreSQL MCP Server

**Use for**: ad-hoc queries and schema inspection against the yield-optimizer Postgres (e.g. `mm_execution_events`, migrations sanity checks). Treat write access carefully on shared or production databases.

**Configuration**: `.cursor/mcp.json` entry `postgres` runs `./scripts/postgres-mcp-wrapper.sh`, which loads repo-root `.env` and passes `MCP_POSTGRES_URL` or `DATABASE_URL` to `npx @modelcontextprotocol/server-postgres`.

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
