## code-weaver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code Weaver is a context management tool for LLMs that helps users gather, manage, and deliver context from codebases to language models. It consists of multiple client applications (VS Code extension, browser extension, webview) that communicate through a centralized WebSocket server, with MCP server integration for Claude Desktop.

## Commands

### Development
- `npm run dev` - Start development servers for all packages
- `npm run build` - Build all packages  
- `npm run test` - Run tests for all packages
- `npm run lint` - Lint all packages
- `npm run lint:fix` - Fix linting issues across all packages
- `npm run clean` - Clean build artifacts

### Individual Package Development
Use `nx` to target specific packages:
- `nx run <package-name>:build` - Build specific package
- `nx run <package-name>:test` - Test specific package  
- `nx run <package-name>:lint` - Lint specific package

### Package-Specific Commands
- **webview**: `npm run dev` (Vite dev server)
- **vscode-extension**: `npm run watch` (esbuild with TypeScript watching)
- **context-manager**: Standard TypeScript compilation

## Architecture

### Core Components
- **@codeweaver/core**: Shared TypeScript interfaces and types
- **@codeweaver/context-manager**: Central context management logic with ContextManager class
- **@codeweaver/websocket-server**: Centralized state management and file operations
- **@codeweaver/websocket-client**: Client-side WebSocket communication
- **@codeweaver/mcp-server**: MCP protocol integration for Claude Desktop

### Client Applications  
- **@codeweaver/vscode-extension**: VS Code extension with webview hosting
- **@codeweaver/browser-extension**: Browser extension for web context capture
- **@codeweaver/webview**: React-based UI for context management

### Key Architectural Patterns
- All file system access is routed through the WebSocket server
- Context sources are managed via a unified `ContextSource` interface with specific subtypes (FileSource, DirectorySource, SnippetSource, GroupSource)
- The ContextManager class handles source validation, group resolution, and snippet management
- Real-time state synchronization between clients via WebSocket connections

### Context Source Types
- **FileSource**: Individual files with content and metadata
- **DirectorySource**: Directory structures and file listings  
- **SnippetSource**: Extracted code snippets with parent file references
- **GroupSource**: Collections of related sources for reusable contexts

## Development Notes

### Monorepo Structure
This is an Nx-managed monorepo with TypeScript packages. Each package has its own build configuration but shares base configurations from the root.

### Testing
- Jest is used across all packages
- Each package has its own test configuration inheriting from `jest.base.config.js`
- Tests are located in `tests/` directories within each package

### Code Organization
- Core types are defined in `@codeweaver/core/src/types/`
- Context management logic is centralized in `@codeweaver/context-manager`
- UI components are in the webview package using React
- Extensions use their respective platform APIs (VS Code, WebExtension)

---
> Source: [richmiles/code-weaver](https://github.com/richmiles/code-weaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
