## obsidian-github-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obsidian GitHub MCP is a Model Context Protocol (MCP) server that connects AI assistants to GitHub repositories containing Obsidian vaults. It enables seamless access to your knowledge base stored on GitHub, allowing AI assistants to read, search, and analyze your Obsidian notes and documentation.

## Key Commands

### Development
- `pnpm install` - Install dependencies
- `pnpm run build` - Build the project (includes linting and formatting)
- `pnpm run dev` - Run in development mode with inspector
- `pnpm run watch` - Watch mode for automatic recompilation
- `pnpm run debug:watch` - Debug mode with watch

### Code Quality
- `pnpm run lint:fix` - Fix linting issues with Biome
- `pnpm run format:fix` - Format code with Biome
- `pnpm run clean` - Clean build artifacts

### Debugging
- `pnpm run inspector` - Launch MCP inspector for testing tools
- `pnpm run logs` - View last 20 lines of MCP logs
- `pnpm run debug` - Run with Node.js debugger attached

## Architecture Overview

The project follows a clean modular architecture:

### Core Components
- **`src/index.ts`**: MCP server initialization using stdio transport. Handles server lifecycle, environment configuration, and graceful shutdown.
- **`src/github/client.ts`**: Encapsulates all GitHub API interactions via Octokit. Implements centralized error handling and tool registration.
- **`src/github/types.ts`**: TypeScript type definitions for configuration.

### Available MCP Tools
1. **`getFileContents`**: Retrieves contents of specific notes, documents, or files from your Obsidian vault
2. **`searchFiles`**: Searches for notes and documents within your vault using GitHub's search syntax (paginated)
3. **`searchIssues`**: Searches for issues and discussions related to your knowledge base projects
4. **`getCommitHistory`**: Tracks how your knowledge base has evolved over time with detailed commit history and diffs

### Key Design Patterns
- **Obsidian-focused design**: Optimized for accessing GitHub repositories containing Obsidian vaults
- **Environment-based configuration**: Repository details via `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`
- **Fail-fast initialization**: Server won't start without required configuration
- **Centralized error handling**: All GitHub requests go through `handleRequest` wrapper
- **Type safety**: Zod schemas for runtime validation of tool inputs
- **Pagination support**: Both searchFiles and getCommitHistory support pagination

### Development Notes
- Uses Biome for linting and formatting (configured in `biome.json`)
- TypeScript target: ES2020 with Node16 module resolution
- Pre-build hooks ensure code quality before compilation
- MCP Inspector available for testing tool interactions
- No test framework currently configured (documentation mentions Jest but not implemented)

---
> Source: [Hint-Services/obsidian-github-mcp](https://github.com/Hint-Services/obsidian-github-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
