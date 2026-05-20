## aws-s3-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building and Running
```bash
npm run build                    # Compile TypeScript to dist/
npm start                       # Run with default transport (STDIO)
node dist/index.js              # Direct execution (equivalent to npm start)
node dist/index.js --http       # Run with HTTP transport on port 3000
node dist/index.js --stdio      # Run with STDIO transport (explicit)
PORT=8080 node dist/index.js --http  # HTTP transport on custom port
```

### Testing
```bash
npm test                        # Run all tests once
npm run test:watch              # Run tests in watch mode
npm run test:coverage           # Run tests with coverage report
npx vitest run src/tools/       # Run specific test directory
npx vitest run --grep "HTTP"    # Run tests matching pattern
```

### Code Quality
```bash
npm run lint                    # Lint code with Biome
npm run format                  # Format code with Biome
npm run check:format            # Check formatting without modifying files
npx @biomejs/biome check --fix --unsafe  # Fix import organization
```

### Docker Development
```bash
npm run docker:build           # Build Docker image
npm run docker:run             # Run container locally (HTTP mode)
docker compose up -d            # Start with MinIO for testing
sh run-inspector.sh             # Debug with MCP Inspector (auto-detects transport)
sh run-inspector.sh --docker --stdio   # Debug in Docker with STDIO
sh run-inspector.sh --docker --http    # Debug in Docker with HTTP
```

## Architecture Overview

This is an AWS S3 Model Context Protocol (MCP) server with a modular architecture supporting multiple transport protocols (STDIO, HTTP, StreamableHTTP).

### Modular Transport Architecture

**Application Layer (`src/application.ts`)**
- Main Application class with dependency injection and configuration management
- Handles CLI argument parsing and environment setup
- Orchestrates transport factory creation and server initialization

**Entry Point (`src/index.ts`)**
- Minimal bootstrap that creates Application instance with parsed CLI arguments
- Supports `--stdio`, `--http`, and transport auto-detection based on environment
- Handles process exit and error scenarios gracefully

**Transport Abstraction (`src/transports/`)**
- `ITransport` interface provides consistent contract for all transport types
- `ITransportFactory` enables runtime transport selection via factory pattern
- `StdioTransport`: Direct process communication (default for Claude Desktop)
- `HttpTransport`: REST API + Server-Sent Events with session management
- Supports both StreamableHTTP and traditional HTTP with proper MCP protocol compliance

**MCP Server (`src/server.ts`)**
- Creates McpServer instance with S3 tools registration
- Transport-agnostic server configuration
- Tool registration via factory pattern from `src/tools/index.ts`

### Core Components

**S3 Resource Layer (`src/resources/s3.ts`)**
- Centralized AWS S3 client management and configuration
- Bucket access validation and filtering based on S3_BUCKETS environment variable
- Provides core S3 operations with error handling and type safety

**Tools Layer (`src/tools/`)**
- Individual MCP tools implementing IMCPTool interface
- Each tool (ListBuckets, ListObjects, GetObject) wraps S3Resource operations
- Consistent error handling and response formatting across all tools

**HTTP Mock Utilities (`src/utils/httpMock.ts`)**
- Bridges Hono HTTP context to Node.js IncomingMessage/ServerResponse
- Enables StreamableHTTPServerTransport compatibility with Hono framework
- Handles session management and streaming response creation

### Key Design Patterns

- **SOLID Principles**: Single responsibility, open/closed, dependency inversion throughout
- **Factory Pattern**: Transport and tool creation through factory functions
- **Dependency Injection**: Application class receives configuration, transport injection
- **Transport Abstraction**: Unified interface supporting STDIO, HTTP, and streaming protocols
- **Session Management**: HTTP transport maintains stateful MCP sessions with proper initialization

### Critical Environment Variables

- `S3_BUCKETS`: Comma-separated list of allowed buckets (security control)
- `AWS_REGION`: AWS region for S3 operations
- `S3_MAX_BUCKETS`: Limits bucket listing results
- `PORT`: HTTP server port (default: 3000)
- `MCP_TRANSPORT`: Force specific transport type ("stdio" or "http")

### Protocol Compliance

**MCP Session Lifecycle**: HTTP transport requires proper session initialization before tool calls
- Clients must send `initialize` request before `tools/list` or `tools/call`
- Sessions are identified by `sessionId` query parameter or `mcp-session-id` header
- StreamableHTTPServerTransport validates Accept headers: requires both `application/json` and `text/event-stream`

### Testing Architecture

- **Unit Tests**: Mocked AWS SDK clients with Vitest
- **Integration Tests**: Real HTTP server startup with session management testing
- **SSE Testing**: TransformStream and streaming response validation
- **Docker Integration**: MinIO compatibility testing via Docker Compose

---
> Source: [samuraikun/aws-s3-mcp](https://github.com/samuraikun/aws-s3-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
