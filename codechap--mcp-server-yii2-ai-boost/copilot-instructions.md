## mcp-server-yii2-ai-boost

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Yii2 AI Boost** is a Model Context Protocol (MCP) server that integrates with Yii2 applications to provide AI assistants with tools for framework introspection, database inspection, and application guidelines. It implements MCP v2025-11-25 with JSON-RPC 2.0 over STDIO transport.

The package is installable as a Composer dependency and provides:
- **16 Tools** for introspection (application info, database schema, database query, config, routes, components, logs, semantic search, model inspector, validation rules, console command inspector, migration inspector, widget inspector, performance profiler, tinker, env inspector)
- **Installation Wizard** for IDE integration (Claude Code, VS Code, Cursor, PhpStorm)
- **Comprehensive Logging** across multiple levels (startup, requests, errors, transport)

## Development Commands

```bash
# Run all tests
composer test

# Generate code coverage report
composer test:coverage

# Check PSR-12 code style
composer cs-check

# Auto-fix code style
composer cs-fix

# Run PHPStan static analysis (level 8)
composer analyze

# Start MCP server (for manual testing)
php yii boost/mcp

# View installation status
php yii boost/info

# Run installation wizard
php yii boost/install

# Update guidelines
php yii boost/update
```

## High-Level Architecture

The codebase follows a **layered, modular design** with clear separation of concerns:

```
CLI Commands Layer
    ↓ (Yii2 Bootstrap)
MCP Server Layer (JSON-RPC dispatcher)
    ├─ Tools (Domain logic - introspection)
    └─ Transports (I/O protocol)
        ↓
Yii2 Application Integration
    └─ Database, Config, Routes, Components
```

### Core Layers

1. **Bootstrap/Commands Layer** (`src/Commands/`, `src/Bootstrap.php`)
   - Entry points via Yii2 console commands
   - `BoostController` - Main dispatcher
   - `McpController` - Starts MCP server with proper logging setup
   - `InstallController` - Installation wizard
   - `InfoController` - Status display
   - `UpdateController` - Guidelines management

2. **MCP Server Layer** (`src/Mcp/Server.php`)
   - JSON-RPC 2.0 protocol handler
   - Dispatches requests to tools/resources
   - Manages tool and resource registration
   - Handles error responses and logging

3. **Tools Layer** (`src/Mcp/Tools/`)
   - Independent, pluggable introspection tools
   - All extend `BaseTool` for consistency
   - Automatic sanitization of sensitive data
   - Support JSON Schema input validation
   - Current tools: ApplicationInfo, DatabaseSchema, DatabaseQuery, ConfigAccess, RouteInspector, ComponentInspector, LogInspector, SemanticSearch, ModelInspector, ValidationRules, ConsoleCommandInspector, MigrationInspector, WidgetInspector, PerformanceProfiler, Tinker, EnvInspector

5. **Search Layer** (`src/Mcp/Search/`)
   - FTS5-powered search using separate SQLite database
   - `MarkdownSectionParser` - Splits markdown into H2 sections
   - `SearchIndexManager` - FTS5 index management (raw PDO, not Yii2 DB)
   - `GitHubGuideDownloader` - Fetches Yii2 guide from GitHub, caches locally
   - Index location: `@runtime/boost/search.db`

4. **Transport Layer** (`src/Mcp/Transports/StdioTransport.php`)
   - STDIO communication (reads STDIN, writes STDOUT)
   - Completely decoupled from business logic
   - Currently STDIO-only (no transport abstraction layer)

### Critical Architectural Patterns

**Plugin Registration Pattern**: Tools are explicitly registered in `Server::registerTools()` - easy to add new tools without modifying core logic.

**Dispatch/Router Pattern**: `Server::dispatch()` routes JSON-RPC method calls to specific handlers. Each method has a dedicated handler function.

**Template Method Pattern**: `BaseTool` provides common functionality (data sanitization, database discovery, logging) that all tools inherit.

**Callback/Handler Pattern**: Transport uses callbacks to decouple I/O from business logic.

## Request Flow (STDIN → STDOUT)

```
1. Client sends JSON-RPC request to STDIN
2. StdioTransport::listen() reads line via fgets()
3. Server::handleRequest() parses and validates JSON
4. Request logged to mcp-requests.log
5. Server::dispatch() routes method to handler
6. Handler executes tool or reads resource
7. BaseTool::sanitize() removes sensitive data
8. Response formatted as JSON-RPC 2.0
9. Response logged to mcp-requests.log
10. Response written to STDOUT
```

**Key Detail**: STDOUT reserved for JSON-RPC responses only. All logging/errors go to STDERR and log files to prevent mixing protocol messages with debug output.

## Adding New Tools

1. Create class in `src/Mcp/Tools/MyTool.php` extending `BaseTool`
2. Implement: `getName()`, `getDescription()`, `getInputSchema()`, `execute()`
3. Add class to `$toolClasses` array in `Server::registerTools()`
4. Tool auto-available via MCP protocol - no other changes needed
5. Use `sanitize()` method for sensitive data
6. Update README and `CLAUDE.md`

## Configuration Files

**`.mcp.json`** - IDE integration (auto-generated by installer)
- Specifies PHP command and arguments for IDE
- Created for Claude Code, VS Code, Cursor, PhpStorm

**`CLAUDE.md`** - Application guidelines wrapper (auto-generated by installer)
- Includes framework and ecosystem guidelines
- Generated fresh with framework instructions

**Files checked into git:**
- `composer.json`, `phpunit.xml`, `phpstan.neon` - Development config
- `src/` - Source code
- `tests/` - Unit tests
- `README.md` - Public documentation

**Files generated at install (not checked in):**
- `.mcp.json`, `CLAUDE.md` - Generated per application
- `.ai/guidelines/` - Downloaded guideline files

## Testing Strategy

**Test Organization**: Two test suites in `tests/` directory:
- **Unit Tests** (`tests/` excluding `tests/Mcp/Tools/`) - Protocol, server, transport tests (no Yii2 context)
- **Tool Tests** (`tests/Mcp/Tools/`) - Tool integration tests using Yii2 bootstrap with SQLite in-memory DB

**Test Scope**: Tests focus on:
- JSON-RPC protocol compliance (`JsonRpcProtocolTest.php`)
- Server request/response structure (`ServerTest.php`)
- Transport layer handling (`StdioTransportTest.php`)
- Markdown section parsing (`Mcp/Search/MarkdownSectionParserTest.php`)
- FTS5 search index management (`Mcp/Search/SearchIndexManagerTest.php`)
- GitHub guide downloading (`Mcp/Search/GitHubGuideDownloaderTest.php`)
- Semantic Search tool execution (`Mcp/Tools/SemanticSearchToolTest.php`)
- Model Inspector tool execution (`Mcp/Tools/ModelInspectorToolTest.php`)
- Validation Rules tool execution (`Mcp/Tools/ValidationRulesToolTest.php`)
- Migration Inspector tool execution (`Mcp/Tools/MigrationInspectorToolTest.php`)
- Widget Inspector tool execution (`Mcp/Tools/WidgetInspectorToolTest.php`)
- Performance Profiler tool execution (`Mcp/Tools/PerformanceProfilerToolTest.php`)
- Tinker tool execution (`Mcp/Tools/TinkerToolTest.php`)
- Environment Inspector tool execution (`Mcp/Tools/EnvInspectorToolTest.php`)

**Tool Test Infrastructure**:
- `tests/yii_bootstrap.php` - Boots a minimal Yii2 console app with SQLite in-memory DB
- `tests/fixtures/SchemaSetupTrait.php` - Creates/drops test tables (user, post, category)
- `tests/fixtures/app/models/` - Fixture ActiveRecord models (User, Post, Category)
- `tests/fixtures/app/widgets/` - Fixture Widget classes (TestWidget)
- `tests/fixtures/github-api-response.json` - Mock GitHub API response for downloader tests
- `tests/Mcp/Tools/ToolTestCase.php` - Base test case with schema setup/teardown

**Limitations**: Full integration testing requires running MCP server with real Yii2 app. Unit tests cannot fully test:
- Transport with actual STDIN/STDOUT
- Real application model discovery (uses fixture models instead)

**Manual Testing**:
```bash
php yii boost/mcp &
# Send JSON-RPC request to stdin, observe response on stdout
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | nc localhost 5000
```

## Logging Strategy

Server logs at multiple levels for different debugging purposes:

1. **mcp-startup.log** (`@runtime/logs/mcp-startup.log`)
   - Server initialization events
   - Working directory, PHP SAPI, environment
   - Tool registration details

2. **mcp-errors.log** (`@runtime/logs/mcp-errors.log`)
   - PHP errors and exceptions
   - Set via `ini_set('error_log', ...)`
   - Custom error handler captures all E_* levels

3. **mcp-requests.log** (`@runtime/logs/mcp-requests.log`)
   - All JSON-RPC requests and responses
   - Protocol traffic tracing
   - Logged before and after request processing

4. **mcp-transport.log** (`/tmp/mcp-server/mcp-transport.log`)
   - Low-level STDIO stream debugging
   - Request/response previews
   - Transport errors and exceptions

All logging goes to STDERR immediately and to files asynchronously. This ensures:
- STDOUT remains clean for JSON-RPC protocol
- Real-time error visibility via STDERR
- Persistent logging to files for analysis
- Multiple tools can tail logs during development

## Important Design Decisions

**Why STDIO-only (not HTTP)?**
- Simplicity for IDE integration
- No network configuration needed
- Security (localhost-only, no port conflicts)
- Sufficient for the primary use case (local IDE tooling)

**Why JSON-RPC 2.0?**
- Official MCP protocol standard
- Better error handling than 1.0
- Notification support (one-way messages)
- Industry standard for RPC

**Why explicit tool registration (not auto-discovery)?**
- Clear control over available tools
- Easier to conditionally enable/disable per deployment
- No file system scanning overhead
- Explicit is better than implicit

**Why separate sanitization?**
- Security by default in all tools
- Prevents accidental credential leaks
- Recursive traversal catches nested sensitive keys
- Patterns are configurable in `BaseTool`

**Why Yii2 Component-based?**
- Dependency injection support
- Configuration via property assignment
- Consistent with Yii2 ecosystem
- Event system available for future use

**Why exceptions convert to JSON-RPC errors?**
- Graceful degradation - client always gets valid JSON
- Error details logged server-side
- No fatal PHP errors exposed to client
- Connection stays open for next request

## Code Style and Quality

- **PSR-12 Compliance**: Checked via phpcs, auto-fixable with `composer cs-fix`
- **Static Analysis**: PHPStan level 8 with `composer analyze`
- **Type Declarations**: Strict types enabled (`declare(strict_types=1)`)
- **Test Coverage**: Aimed at protocol and core logic (not full integration)

## Key Files and Their Roles

- **`src/Mcp/Server.php`** - Core orchestrator, protocol handler, tool/resource registry
- **`src/Mcp/Tools/Base/BaseTool.php`** - Base class providing sanitization, DB discovery, model resolution
- **`src/Mcp/Transports/StdioTransport.php`** - STDIO I/O handler
- **`src/Bootstrap.php`** - Yii2 integration entry point
- **`src/Commands/McpController.php`** - MCP server starter with logging setup
- **`src/Commands/InstallController.php`** - Installation wizard creating config files
- **`src/Mcp/Search/MarkdownSectionParser.php`** - Markdown to section splitter
- **`src/Mcp/Search/SearchIndexManager.php`** - FTS5 index (raw PDO to SQLite)
- **`src/Mcp/Search/GitHubGuideDownloader.php`** - Yii2 guide fetcher/cache
- **`tests/JsonRpcProtocolTest.php`** - Protocol compliance tests
- **`phpstan.neon`** - Static analysis config (level 8 via composer script)
- **`phpunit.xml`** - Test suite config with coverage reporting

## Common Development Tasks

**Investigating a Tool Failure**:
1. Check `@runtime/logs/mcp-errors.log` for PHP errors
2. Check `@runtime/logs/mcp-requests.log` for the request that failed
3. Check `@runtime/logs/mcp-startup.log` to confirm tool registered
4. Run tool's `execute()` method directly with test data if possible

**Adding Debug Output**:
- Use Server::log() method - it logs to mcp-startup.log
- Do not echo/var_dump - will corrupt STDOUT JSON

**Debugging Protocol Issues**:
- Check `@runtime/logs/mcp-requests.log` for malformed responses
- Check `/tmp/mcp-server/mcp-transport.log` for STDIO issues
- Compare requests/responses against JSON-RPC 2.0 spec

**Testing a New Tool**:
```bash
php yii boost/mcp &
# In another terminal, send:
echo '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"my_tool","arguments":{"key":"value"}}}' | ...
```

## Deployment Notes

1. **Install**: Run `php yii boost/install` to generate config files
2. **Verify**: Run `php yii boost/info` to confirm installation
3. **IDE Setup**: Point IDE to generated `.mcp.json` file
4. **Logs**: Monitor `@runtime/logs/` directory during development
5. **PHP Version**: Requires PHP 7.4+ (type hints, null safe operator in Yii2)
6. **Permissions**: STDOUT/STDERR must be writable, `@runtime/logs/` must be writable

## Future Expansion Points

**Phase 2 Tools** (complete):
- `model_inspector` - Active Record model analysis (attributes, relations, behaviors, scenarios, fields)
- `validation_rules` - Model validation introspection (rules, messages, constraints, safe attributes)

**Phase 3 Tools** (complete):
- `migration_inspector` - Migration status, history, pending, source viewing
- `widget_inspector` - Widget discovery, properties, methods, events, hierarchy
- `performance_profiler` - EXPLAIN plans, index analysis, missing index detection

**Phase 4 Tools** (complete):
- `tinker` - Execute arbitrary PHP code in Yii2 application context
- `env_inspector` - Environment variables, PHP extensions, system configuration

**Phase 5 — Semantic Search** (complete):
- `semantic_search` - FTS5-powered search replacing grep-based `search_guidelines`
- `MarkdownSectionParser` - Section-level indexing (H2 boundaries)
- `SearchIndexManager` - SQLite FTS5 with BM25 ranking
- `GitHubGuideDownloader` - Yii2 definitive guide from GitHub, cached locally
- Index: bundled guidelines (~60 sections) + Yii2 guide (~500+ sections)

**Transport Expansion** (would require transport abstraction layer):
- `HttpTransport` - For non-IDE MCP clients
- `WebSocketTransport` - For real-time communication

All of these follow existing patterns and can be added without modifying core architecture.

---
> Source: [codeChap/mcp-server-yii2-ai-boost](https://github.com/codeChap/mcp-server-yii2-ai-boost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
