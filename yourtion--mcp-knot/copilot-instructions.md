## mcp-knot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development

- `pnpm dev:api` - Start API server in development mode with hot reload
- `pnpm dev:fe` - Start frontend in development mode
- `pnpm dev:cli` - Start CLI package in development mode
- `pnpm build` - Build all packages
- `pnpm build:production` - Build all packages for production deployment

### Testing & Quality

- `pnpm test` - Run all tests (uses vitest 4.x with projects)
- `pnpm test:watch` - Run tests in watch mode
- `pnpm test:e2e` - Run E2E tests only
- `pnpm test:coverage` - Run tests with coverage reports
- `pnpm test:debug` - Run tests in debug mode (VITEST_DEBUG=true)
- `pnpm check` - Run linting with oxlint and formatting with oxfmt
- `pnpm check:all` - Run oxlint and oxfmt on entire codebase

### Package-specific Commands

- `pnpm --filter @mcp-core/mcp-knot-core dev` - Watch mode compilation for core package
- `pnpm --filter @mcp-core/mcp-knot-cli test:e2e` - End-to-end tests for CLI package
- `pnpm --filter @mcp-core/mcp-knot-api test:mcp` - MCP protocol tests for API package

### Deployment

- `pnpm start:api` - Start API server in production
- `pnpm start:cli` - Start CLI server in production

## Architecture Overview

MCP Knot is a modular monorepo that provides a centralized hub for managing multiple MCP (Model Context Protocol) servers through different interfaces.

### Core Architecture

The project follows a modular architecture with clear separation of concerns:

1. **Core Package** (`packages/core/`) - Provides the foundational MCP service management, connection handling, and tool execution capabilities
2. **API Package** (`backend/`) - Web API server built with Hono that exposes HTTP/SSE endpoints
3. **CLI Package** (`packages/cli/`) - Standalone MCP server that communicates via stdin/stdout
4. **Share Package** (`packages/share/`) - Shared types and utilities
5. **Frontend** (`frontend/`) - Vue.js web interface for management

### Key Design Patterns

#### Service Layer Architecture

- **MCP Service Manager**: Centralized management of MCP server connections
- **Server Connection Manager**: Handles connection pooling and lifecycle management
- **Tool Registry**: Manages tool discovery, validation, and execution
- **Group Manager**: Handles group-based routing and access control

#### Configuration System

- **Multi-layer Configuration**: Supports both legacy config files and new validated configuration
- **Environment-based Overrides**: Supports environment variable overrides for deployment
- **Validation**: Uses Zod schemas for comprehensive configuration validation

#### Error Handling

- **Unified Error System**: Centralized error handling with proper error codes and categories
- **Graceful Degradation**: Services continue operating even when some MCP servers fail
- **Error Recovery**: Automatic reconnection and retry mechanisms

### Package Structure

```
packages/
├── core/                    # @mcp-core/mcp-knot-core
│   ├── src/
│   │   ├── api-to-mcp/     # API to MCP conversion services
│   │   ├── config/         # Configuration management
│   │   ├── errors/         # Error handling system
│   │   ├── services/       # Core services (MCP, tools, connections)
│   │   ├── types/          # Type definitions
│   │   └── utils/          # Utility functions
├── cli/                     # @mcp-core/mcp-knot-cli
│   ├── src/
│   │   ├── config/         # CLI-specific configuration
│   │   ├── protocol/       # MCP protocol handling
│   │   ├── server/         # CLI server implementation
│   │   ├── transport/      # Transport layer (stdin/stdout)
│   │   └── utils/          # CLI utilities
└── share/                   # @mcp-core/mcp-knot-share
    └── src/
        ├── types/          # Shared types
        └── utils/          # Shared utilities
```

### API Endpoints Structure

The API server provides two main endpoint patterns:

1. **Global Endpoint** (`/mcp`) - Legacy endpoint for all tools
2. **Group-based Endpoints** (`/:group/mcp`) - Group-specific endpoints with access control

Key API routes:

- `/api/groups` - Group management
- `/api/servers` - MCP server management
- `/api/tools` - Tool discovery and testing
- `/api/auth` - Authentication endpoints

### MCP Protocol Integration

The project implements the Model Context Protocol in multiple ways:

1. **HTTP/SSE Transport**: For web-based clients with streaming capabilities
2. **CLI Transport**: For integration with MCP clients like Claude Desktop
3. **Internal Transport**: For inter-service communication

### Authentication & Security

- **JWT-based Authentication**: Token-based authentication for web interface
- **Group-level Security**: Each group can have separate validation keys
- **Tool Filtering**: Groups support allow/deny lists for tools
- **Rate Limiting**: Built-in rate limiting for API endpoints

### Configuration Management

Configuration is handled through multiple layers:

1. **MCP Service Configuration** (`mcp_server.json`) - Defines MCP servers
2. **Group Configuration** (`group.json`) - Defines server groups and access rules
3. **System Configuration** - General system settings
4. **Environment Variables** - Override any configuration value

### Testing Strategy

The project uses Vitest 4.x with a centralized configuration:

- **Root Config**: `vitest.config.ts` defines all projects (share/core/cli/api-unit/api-integration/api-e2e/frontend)
- **TestContext**: `test/context/test-context.ts` provides resource lifecycle management via `useTestContext()`
- **Unit Tests**: Individual component testing (`.unit.test.ts` suffix)
- **Integration Tests**: Service interaction testing
- **E2E Tests**: Full MCP protocol testing with mock servers (6 files in `backend/src/e2e/`)
- **Coverage Reports**: Code coverage tracking with V8 provider

Testing conventions:

- Use `function` keyword (not arrow functions) for `vi.fn().mockImplementation()` to support `new` calls
- Use `useTestContext()` for managing test resources that need cleanup
- Never use `expect(true).toBe(true)` - always assert actual behavior
- Never use `any` type in test files

## 开发习惯（必须遵循）

- 任务完成后需要执行 `pnpm test` 取保测试全部通过，确保新代码不破坏原有逻辑
- 同时对新开发的代码编写相应的测试，保证代码的正确性
- 再执行 `pnpm check`和`pnpm build`保证代码质量，执行后需要再运行 `pnpm test` 保证测试通过
- **提交/合并前必须跑 `pnpm check:ci`（CI 实际门禁，readonly）**：它跑 `oxlint` + `oxfmt --check`，与 `pnpm check`（含 `--fix`，会改写文件）不同。`pnpm check` 修完后仍要再用 `pnpm check:ci` 复验，确保没有遗漏的格式/lint 问题进入主干
- 代码中不要使用 `any`类型！！！通过泛型或`Record`确保类型正确
- 任务或者需求完成后，需要检查是否需要更新 README 或者其他相关的文档，确保文档的信息有效，并于代码同步

### Package Dependencies

The project uses workspace dependencies with clear boundaries:

- **core** depends on **share**
- **cli** depends on **core** and **share**
- **api** depends on **core** and **share**
- **frontend** depends on **share**

This ensures proper dependency management and avoids circular dependencies.

---
> Source: [yourtion/mcp-knot](https://github.com/yourtion/mcp-knot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
