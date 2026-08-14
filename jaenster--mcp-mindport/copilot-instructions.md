## mcp-mindport

> This is a high-performance Model Context Protocol (MCP) server built in TypeScript/Node.js that provides optimized storage and search capabilities for AI systems.

# MindPort MCP Server

This is a high-performance Model Context Protocol (MCP) server built in TypeScript/Node.js that provides optimized storage and search capabilities for AI systems.

## Project Overview

MindPort is designed to work seamlessly with Claude Code CLI and other MCP clients via standard stdio JSON-RPC, offering:

- **Standard MCP Protocol**: Communicates via stdio (no WebSockets required)
- **Optimized Search**: Token-efficient search designed for AI interactions
- **Domain Management**: Organize resources by project or context
- **Resource Storage**: Store code, documentation, and other content with metadata
- **Prompt Templates**: Manage reusable prompt templates
- **Full-text Search**: Powered by Bleve search engine
- **Fast Storage**: BadgerDB key-value store for performance
- **Optional Daemon Mode**: WebSocket support for advanced use cases

## Development Commands

```bash
# Install/setup for Claude Code
./claude-code-setup.sh

# Full installation with Claude Desktop support
./install.sh

# Build the binary
npm install

# Run tests
npm test
./run_tests.sh

# Run integration tests
go test ./tests/

# Run in stdio mode (default MCP transport)
node src/index.js

# Start in specific domain (stdio mode)
node src/index.js --domain project1

# Run with custom config (stdio mode)
node src/index.js --config /path/to/config.yaml

# Logging is automatically disabled in stdio mode (default behavior)
node src/index.js

# Enable logging to file in stdio mode (if needed for debugging)
node src/index.js --log /path/to/mindport.log

# Optional: Start as daemon (WebSocket mode)
node src/index.js --daemon
```

## Key Files

- `main.go` - CLI entry point and configuration
- `internal/mcp/server.go` - MCP protocol implementation
- `internal/storage/` - Storage layer (BadgerDB)
- `internal/search/` - Search engine (Bleve)
- `internal/domain/` - Domain management
- `tests/` - Integration and unit tests

## Configuration

Default config at `~/.config/mindport/config.yaml`:

```yaml
server:
  host: "localhost"
  port: 8080

storage:
  path: "~/.config/mindport/data/storage"

search:
  index_path: "~/.config/mindport/data/search"

domain:
  default_domain: "default"

daemon:
  pid_file: "/tmp/mcp-mindport.pid"
  log_file: "~/.config/mindport/mindport.log"
```

## Available MCP Tools

When running as an MCP server, MindPort provides these tools:

### Resource Management
- `store_resource` - Store content with metadata and tags
- `search_resources` - Fast, token-efficient search
- `advanced_search` - Complex queries with filters
- `get_resource` - Retrieve specific resources

### Search Tools
- `grep` - Pattern matching in stored content
- `find` - Find resources by name/title patterns
- `ripgrep` - Advanced text search with regex

### Domain Management
- `create_domain` - Create new domain contexts
- `list_domains` - List available domains
- `switch_domain` - Change current domain
- `domain_stats` - Get domain statistics

### Prompt Management
- `store_prompt` - Store reusable prompt templates
- `list_prompts` - List available prompts
- `get_prompt` - Retrieve and render prompts

## Testing

The project includes comprehensive test coverage:

- `tests/domain_integration_test.go` - Domain functionality
- `tests/mcp_integration_test.go` - MCP protocol compliance
- `tests/error_handling_test.go` - Error scenarios
- `tests/workflow_integration_test.go` - End-to-end workflows

Run with: `go test ./tests/ -v`

## Architecture

```
┌─────────────────┐
│   MCP Client    │ (Claude Code, Claude Desktop, etc.)
│                 │
└─────────┬───────┘
          │ JSON-RPC 2.0
          │
┌─────────▼───────┐
│   MCP Server    │ (Protocol handling)
│                 │
├─────────────────┤
│ Domain Manager  │ (Context/project management)
│                 │
├─────────────────┤
│ Search Engine   │ (Bleve full-text search)
│                 │
├─────────────────┤
│ Storage Layer   │ (BadgerDB key-value store)
│                 │
└─────────────────┘
```

## Development Notes

- Built with Go 1.21+
- Uses Cobra for CLI interface
- Primary transport: stdio JSON-RPC (standard MCP)
- Optional WebSocket transport for daemon mode
- Domain-aware resource organization
- Optimized for minimal token usage in responses
- Thread-safe concurrent operations
- Graceful shutdown handling

## Environment Variables

- `MCP_MINDPORT_CONFIG` - Config file path
- `MCP_MINDPORT_DAEMON` - Run as daemon (true/false)
- `MCP_MINDPORT_DOMAIN` - Start in specific domain
- `MCP_MINDPORT_DEFAULT_DOMAIN` - Set default domain
- `MCP_MINDPORT_LOG` - Log file path ('discard' to disable logging)

---
> Source: [jaenster/mcp-mindport](https://github.com/jaenster/mcp-mindport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
