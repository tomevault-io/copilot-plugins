## github-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

This is a GitHub MCP (Model Context Protocol) server that provides GitHub
integration capabilities for Claude and other AI systems. The server exposes
GitHub functionality through the MCP protocol.

## Development Commands

```bash
# Install dependencies
npm install

# Run in development mode
npm run dev

# Build the project
npm run build

# Run tests
npm test

# Code quality
npm run lint              # ESLint check
npm run lint:fix          # ESLint auto-fix
npm run format            # Prettier format
npm run format:check      # Format check
npm run type-check        # TypeScript check

# Security
npm run security:check    # Gitleaks secret detection

# Pre-commit hooks
npm run pre-commit        # Run all pre-commit hooks
```

## Architecture

This MCP server will typically consist of:

- **Server implementation**: Core MCP protocol handling
- **GitHub API integration**: Methods to interact with GitHub's REST/GraphQL
  APIs
- **Tool definitions**: MCP tools that expose GitHub functionality
- **Authentication handling**: GitHub token management and OAuth flows
- **Resource management**: Handling GitHub resources like repositories, issues,
  PRs

## Key Considerations

- GitHub API rate limiting and authentication
- MCP protocol compliance and tool definitions
- Error handling for GitHub API responses
- Secure handling of GitHub tokens and credentials
- Support for both personal and organization repositories

## Testing

Tests should cover:

- MCP protocol compliance
- GitHub API integration
- Authentication flows
- Error handling scenarios
- Rate limiting behavior

---
> Source: [juliar13/github-mcp-server](https://github.com/juliar13/github-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
