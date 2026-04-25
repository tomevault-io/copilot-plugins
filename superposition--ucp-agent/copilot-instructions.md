## ucp-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UCP Agent is a Universal Commerce Protocol implementation with Claude AI integration. It provides:
- A UCP-compliant merchant server with discovery and checkout capabilities
- A Claude-powered shopping assistant agent
- An MCP (Model Context Protocol) server for tool-based commerce interactions

## Commands

```bash
bun install              # Install dependencies
bun run start            # Run the server (or: bun run index.ts)
bun run dev              # Run with watch mode
bun run mcp              # Run standalone MCP server
bun run typecheck        # Type check without emitting
bun test                 # Run tests
bun test path/to/file    # Run a single test file
```

## Environment Variables

- `PORT` - Server port (default: 3000)
- `ANTHROPIC_API_KEY` - Enables Claude agent interactive mode
- `MERCHANT_ENDPOINT` - MCP server's target merchant (default: http://localhost:3000)

## Architecture

```
index.ts                      # Entry point - starts server + optional REPL
src/
  server/ucp-server.ts        # Hono-based UCP merchant server
  agent/claude-agent.ts       # Claude AI shopping assistant
  mcp/ucp-mcp-server.ts       # MCP server with commerce tools
  mcp/run-mcp.ts              # MCP server entry point
  sdk/                        # Zod schemas for UCP types
    schemas/common.ts         # Money, Address, Cart, Customer
    schemas/checkout.ts       # CheckoutSession, payment types
    schemas/discovery.ts      # UCP discovery response types
```

**Key endpoints:**
- `GET /.well-known/ucp` - UCP discovery
- `POST /ucp/checkout` - Create checkout session
- `GET/PATCH /ucp/checkout/:sessionId` - Get/update session

**MCP tools:** `discover_merchant`, `create_checkout`, `get_checkout`, `update_checkout`

## Bun Preferences

Use Bun instead of Node.js:
- `bun <file>` instead of `node` or `ts-node`
- `bun test` instead of jest/vitest
- `bun install` instead of npm/yarn/pnpm
- Bun auto-loads `.env` files

Preferred APIs:
- `Bun.serve()` for HTTP (not express)
- `Bun.file()` over `node:fs` readFile/writeFile
- Built-in `WebSocket` (not ws package)
- `bun:sqlite`, `Bun.redis`, `Bun.sql` for databases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superposition) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
