## tarx-palantir-connector

> 1. Call tarx_session_start(repo="all") FIRST

# TARX Claude Code Session

## Session Start Protocol (MANDATORY)
1. Call tarx_session_start(repo="all") FIRST
2. Read every active_decision before touching anything
3. Call tarx_decision_store() after every significant change
4. Never deploy to saase scope
5. Never revert llama-server to Ollama

## MCP Config
This repo uses TARX MCP at mcp.tarx.com for memory + decisions.
See .mcp.json for full server config.

## Architecture (load from tarx_memory_search)
Search: "tarx:architecture" for Soul/Mind/Synthesis
Search: "tarx:ship-status" for current state
Search: "tarx:cc-protocol" for session rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tarx-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
