## palo-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PanOS-MCP is an MCP (Model Context Protocol) server for interacting with Palo Alto Networks PanOS firewalls.

## Build & Development Commands

```bash
# Install dependencies
npm install

# Build
npm run build

# Watch mode (rebuild on changes)
npm run dev

# Run the server
npm run start
```

## Architecture

<!-- TODO: Document architecture once implemented -->

### MCP Server Structure
- Entry point: `src/index.ts`
- Tools: `src/tools/` - MCP tool implementations for PanOS operations
- Resources: `src/resources/` - MCP resource providers

### PanOS Integration
- API client for PanOS XML API
- Authentication handling
- Firewall configuration operations

## Environment Configuration

<!-- TODO: Document required environment variables -->
```
PANOS_HOST=<firewall-hostname-or-ip>
PANOS_API_KEY=<api-key>
```

## Key Patterns

<!-- TODO: Document project-specific patterns as they emerge -->

---
> Source: [apius-tech/Palo-MCP](https://github.com/apius-tech/Palo-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
