## mcp-grafana

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides tools for interacting with the Grafana API. The server exposes Grafana operations through MCP tools that can be used by Claude and other MCP clients.

## Key Commands

### Development

- `make test` - Run all tests
- `go test ./internal/tools/mapper -v` - Run mapper tests specifically
- `go test ./internal/tools/mapper -v -run TestName` - Run specific test
- `make build` - Build the MCP server binary
- `make fmt` - Format code (runs gofumpt and goimports-reviser)
- `make lint` - Run golangci-lint

### MCP Development

- `make mcp-shell` - Run the MCP server interactively with mcptools shell for testing
- **MCP Server Testing**: Use `make mcp-shell` for interactive testing rather than piping JSON-RPC directly to the server, which will hang waiting for a persistent connection

### CI Commands

- `make test-ci` - Run tests with coverage
- `make lint-ci` - Run linters for CI
- `make format-ci` - Check formatting in CI

## Architecture

### Core Components

**MCP Server Entry Point** (`cmd/server/main.go`)

- Creates MCP server instance using mark3labs/mcp-go
- Registers all MCP components:
  - `prompts.RegisterMCPPrompts()`
  - `resources.RegisterMCPResources()`
  - `tools.RegisterMCPTools()`
- Serves over stdio protocol

**Tool Registration** (`internal/tools/register.go`)

- Central registration point for all MCP tools
- Initializes resource mappers before registering tools

## Detailed Development Guides

For comprehensive development guidance, see the following detailed guides:

@docs/development-best-practices.md

@docs/dependency-management.md

@docs/ci-best-practices.md

@docs/mcp-server-development.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krmcbride)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krmcbride)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
