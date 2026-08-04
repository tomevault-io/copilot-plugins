## love2d-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server for LÖVE2D game development. MCP servers expose tools and resources that allow LLMs to interact with external systems.

## Architecture

### MCP Server Implementation
- The server exposes tools for interacting with a running LÖVE2D game instance
- Communication happens via stdio (standard input/output) following the MCP protocol
- Tools should enable querying game state, modifying entities, and executing Lua code in the LÖVE2D runtime

### LÖVE2D Integration
- LÖVE2D uses Lua as its scripting language
- The game engine provides callbacks like `love.load()`, `love.update(dt)`, `love.draw()`
- Games are typically structured with main.lua as the entry point
- Common modules: graphics, audio, physics, filesystem, keyboard, mouse

## Development Workflow

### Testing
- MCP servers are typically tested using the MCP Inspector tool
- LÖVE2D games can be run with the `love` command pointing to the game directory
- Integration testing requires both the MCP server and a running LÖVE2D instance

### Key Considerations
- The MCP server needs a way to communicate with the LÖVE2D process (socket, shared memory, or file-based communication)
- Lua code execution should be sandboxed appropriately for security
- State queries should be efficient to avoid impacting game performance
- Tool responses should follow MCP protocol specifications

---
> Source: [shayarnett/love2d-mcp](https://github.com/shayarnett/love2d-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
