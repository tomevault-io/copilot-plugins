## mcpcodeserver

> This document provides comprehensive guidance for Claude Code (claude.ai/code) and other AI agents when working with this codebase. It covers project structure, implementation details, development workflows, and testing strategies.

# AGENTS.md - mcpcodeserver Project Structure

This document provides comprehensive guidance for Claude Code (claude.ai/code) and other AI agents when working with this codebase. It covers project structure, implementation details, development workflows, and testing strategies.

## Project Overview

**mcpcodeserver** is an MCP (Model Context Protocol) proxy server that transforms tool calling into code generation. It connects to child MCP servers as a client, discovers their tools, and exposes two meta-tools that allow LLMs to generate and execute TypeScript code that calls those tools.

### Key Innovation

Instead of the traditional pattern:

```
LLM → Tool Call 1 → LLM → Tool Call 2 → LLM → Tool Call 3 → LLM
```

This enables:

```
LLM → Generate Code → Execute Code (calls Tool 1, 2, 3 internally) → Result
```

## Directory Structure

```
mcpcodeserver/
├── src/
│   ├── index.ts           # CLI entry point and configuration loading
│   ├── server.ts          # Main MCP server implementation
│   ├── client-manager.ts  # Manages connections to child MCP servers
│   ├── codegen.ts         # TypeScript code generation from tool schemas
│   ├── sandbox.ts         # VM-based code execution sandbox
│   └── types.ts           # TypeScript type definitions
├── tests/
│   ├── unit/              # Unit tests
│   ├── integration/       # Integration tests
│   ├── vm-tests/          # TypeScript test programs for VM execution
│   └── mock-server/       # Pizza Shop test server (Python)
├── dist/                  # Compiled JavaScript output (generated)
├── node_modules/          # Dependencies (generated)
├── package.json           # Project metadata and dependencies
├── tsconfig.json          # TypeScript compiler configuration
├── mcp.json.example       # Example configuration file
├── README.md              # User documentation
├── AGENTS.md              # This file - developer/agent documentation
└── .gitignore             # Git ignore rules
```

## Using with MCP Clients

**Recommended:** Use `npx` to run mcpcodeserver without installation:

```json
{
  "mcpServers": {
    "codeserver": {
      "command": "npx",
      "args": ["-y", "mcpcodeserver", "--config", "/path/to/mcp.json"]
    }
  }
}
```

**From GitHub directly:**
```json
{
  "mcpServers": {
    "codeserver": {
      "command": "npx",
      "args": ["-y", "github:zbowling/mcpcodeserver", "--config", "/path/to/mcp.json"]
    }
  }
}
```

**Other package managers:** `yarn dlx`, `pnpm dlx`, `bunx` all work similarly.

See [examples/](./examples/) for more configuration options.

## Child Server Configuration Format (mcp.json)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "env": { "DEBUG": "false" }
    },
    "api-server": {
      "url": "http://localhost:3000/mcp",
      "transport": "sse"
    }
  }
}
```

**Transport types:**
- **stdio**: Spawns child process (requires `command`, optional `args` and `env`)
- **sse**: HTTP Server-Sent Events (requires `url`, `transport: "sse"`)

## Module Documentation

### 1. index.ts - CLI Entry Point

**Purpose:** Command-line interface and application bootstrapping

**Key Functions:**

- `parseArgs()` - Parse command-line arguments (--config, --help)
- `showHelpMessage()` - Display usage information
- `loadConfig(configPath)` - Load and validate mcp.json configuration
- `main()` - Main entry point that orchestrates startup

**Flow:**

1. Parse command-line arguments
2. Load configuration from mcp.json
3. Validate configuration structure
4. Start the MCP server via `runServer()`

**Configuration Format:**

```typescript
interface MCPConfig {
  mcpServers: Record<string, MCPServerConfig>
}

interface MCPServerConfig {
  command?: string;      // For stdio transport
  args?: string[];       // Command arguments
  env?: Record<string, string>;  // Environment variables
  url?: string;          // For HTTP/SSE transport
  transport?: "stdio" | "sse";   // Transport type
}
```

### Tool Naming Convention

- **Discovery:** Child tools are cached as `serverName.toolName` (e.g., `filesystem.read_file`)
- **Generated functions:** Converted to `serverName_toolName` (e.g., `filesystem_read_file()`)
- This prevents naming conflicts when multiple servers expose similar tools

### Data Flow: execute_toolcall_script

1. User calls `execute_toolcall_script({ code: "...", timeout: 30000 })`
2. Server validates code and timeout parameters
3. `executeSandbox()` creates VM context with injected `__callTool()` function
4. `generateRuntimeStubs()` creates tool function wrappers
5. Code executed: `(async () => { stubs + userCode })()`
6. User code calls `await filesystem_read_file({ path: "..." })`
7. Stub calls `__callTool("filesystem.read_file", params)`
8. `clientManager.callTool()` routes to appropriate child server
9. Result returned to user code, which continues execution
10. Final result, console logs, and errors formatted and returned

### 2. server.ts - MCP Server Implementation

**Purpose:** Main MCP server that exposes tools to parent clients

**Key Components:**

- `createServer(config)` - Creates and configures the MCP server
- `runServer(config)` - Starts the server with stdio transport

**Exposed Tools:**

#### Tool 1: get_tool_definitions

- **Input:** `{ include_examples?: boolean }`
- **Output:** TypeScript code with type definitions
- **Process:**
  1. Get all tools from client manager
  2. Generate TypeScript interfaces and function declarations
  3. Optionally include usage examples
  4. Return as text content

#### Tool 2: execute_toolcall_script

- **Input:** `{ code: string, timeout?: number }`
- **Output:** Execution result with console logs and return value
- **Process:**
  1. Validate code parameter
  2. Set timeout (default 30s, max 5min)
  3. Execute code in sandbox with tool access
  4. Format and return result

**Request Handlers:**

- `tools/list` - Returns the two meta-tools
- `tools/call` - Handles execution of get_tool_definitions or execute_toolcall_script

**Signal Handling:**

- SIGINT and SIGTERM trigger graceful shutdown
- Disconnects all child clients before exit

### 3. client-manager.ts - Child Server Management

**Purpose:** Manages connections to multiple child MCP servers

**Key Class:** `MCPClientManager`

**Responsibilities:**

1. Connect to child MCP servers (stdio or HTTP/SSE)
2. Discover tools from each server
3. Maintain a cache of available tools
4. Route tool calls to appropriate child server
5. Handle errors and connection lifecycle

**Key Methods:**

#### `connect(): Promise<void>`

- Connects to all servers defined in config
- Creates appropriate transport (stdio or SSE)
- Discovers all available tools
- Populates tools cache

#### `connectToServer(serverName, config): Promise<void>`

- Connects to a single child server
- Determines transport type from config
- For stdio: spawns child process with command/args/env
- For SSE: connects to HTTP endpoint

#### `discoverTools(): Promise<void>`

- Queries each connected server for its tools via `listTools()`
- Stores tools with fully qualified names: `serverName.toolName`
- Example: `filesystem.read_file`, `github.create_issue`

#### `callTool(qualifiedToolName, args): Promise<ToolResult>`

- Parses qualified name into server and tool name
- Routes call to appropriate client
- Returns result or throws ToolExecutionError
- Wraps errors with context (server name, tool name)

#### `getTools(): DiscoveredTool[]`

- Returns array of all discovered tools with metadata
- Used by codegen to generate TypeScript definitions

#### `disconnect(): Promise<void>`

- Closes all client connections
- Clears internal caches
- Called during graceful shutdown

**Tool Naming Convention:**

- Internal storage: `serverName.toolName` (e.g., `filesystem.read_file`)
- Generated function names: `serverName_toolName` (e.g., `filesystem_read_file`)
- This avoids naming conflicts between servers

### 4. codegen.ts - TypeScript Code Generation

**Purpose:** Convert JSON Schema tool definitions to TypeScript

**Key Functions:**

#### `generateTypeDefinitions(tools): string`

- **Input:** Array of discovered tools with JSON Schema
- **Output:** TypeScript code with interfaces and function declarations
- **Process:**
  1. Generate ToolResult interface (common return type)
  2. For each tool:
     - Convert JSON Schema to TypeScript interface
     - Generate function declaration with JSDoc
     - Include server and tool name in comments

**Example Output:**

```typescript
interface ReadFileParams {
  path: string;
}

/**
 * Read contents of a file
 * Server: filesystem
 * Tool: read_file
 */
declare function filesystem_read_file(params: ReadFileParams): Promise<ToolResult>;
```

#### `generateRuntimeStubs(tools): string`

- **Input:** Array of discovered tools
- **Output:** JavaScript code with function implementations
- **Process:**
  1. For each tool, create async function
  2. Function calls `__callTool(qualifiedName, params)`
  3. `__callTool` is injected by sandbox and routes to client manager

**Example Output:**

```javascript
async function filesystem_read_file(params) {
  return await __callTool("filesystem.read_file", params);
}
```

#### `jsonSchemaToTypeScript(schema): string`

- Converts JSON Schema types to TypeScript types
- Handles:
  - Primitives: string, number, boolean, null
  - Arrays with item types
  - Objects with properties
  - Enums
  - Union types (anyOf, oneOf)
  - Intersection types (allOf)
  - Optional properties based on `required` field

#### `generateUsageExample(tools): string`

- Creates example code showing how to use tools
- Includes error handling patterns
- Shows a few representative tools

**Helper Functions:**

- `generateInterfaceName(toolName)` - Converts `read_file` to `ReadFileParams`
- `generateFunctionName(qualifiedName)` - Converts `fs.read` to `fs_read`
- `getExampleValue(schema)` - Generates example values for documentation

### 5. sandbox.ts - Code Execution Sandbox

**Purpose:** Execute user-provided TypeScript code safely with tool access

**Key Function:** `executeSandbox(code, clientManager, config): Promise<SandboxResult>`

**Sandbox Configuration:**

```typescript
interface SandboxConfig {
  timeout?: number;        // Max execution time (default 30s)
  allowConsole?: boolean;  // Whether to allow console output
}
```

**Sandbox Result:**

```typescript
interface SandboxResult {
  result: any;            // Return value from code
  logs: string[];         // Console output captured
  error?: {               // Any errors that occurred
    message: string;
    stack?: string;
  };
}
```

**Execution Process:**

1. **Create Sandbox Context:**
   - Provide console methods (log, error, warn, info) that capture output
   - Provide basic JavaScript globals (Math, JSON, Date, etc.)
   - Inject `__callTool(qualifiedName, params)` function
   - NO access to Node.js modules (fs, http, etc.)

2. **Generate Runtime Code:**
   - Generate tool stub functions via `generateRuntimeStubs()`
   - Wrap user code in async IIFE: `(async () => { userCode })();`
   - Combine stubs + user code

3. **Execute with VM:**
   - Create VM context with sandbox globals
   - Compile code as a VM script
   - Execute with timeout protection
   - Capture return value and errors

4. **Return Result:**
   - Format console logs
   - Include return value
   - Include error details if execution failed

**Security Considerations:**

- Not a full security sandbox
- VM context provides isolation but not bulletproof
- No access to require/import
- No access to process or filesystem (except via tools)
- Execution timeout prevents infinite loops
- Only execute trusted code

**Helper Functions:**

- `validateCode(code)` - Basic syntax validation
- `formatSandboxResult(result)` - Pretty-print results with sections for logs, result, errors

### 6. types.ts - Type Definitions

**Purpose:** Centralized TypeScript types for the entire project

**Key Types:**

#### `MCPServerConfig`

Configuration for a single child MCP server

- stdio fields: command, args, env
- SSE fields: url, transport

#### `MCPConfig`

Main configuration file structure

- Contains `mcpServers` dictionary

#### `DiscoveredTool`

Metadata for a tool discovered from a child server

- name, description, inputSchema, serverName

#### `ToolResult`

Result from executing a tool

- content array (text, image, resource)
- isError flag

#### `ToolExecutionError`

Error wrapper with context

- message, toolName, serverName, details

## Data Flow

### Startup Flow

```
1. index.ts:main()
   ↓
2. loadConfig() → Parse mcp.json
   ↓
3. createServer(config)
   ↓
4. MCPClientManager.connect()
   ↓
5. For each server in config:
   - connectToServer()
   - Create transport (stdio or SSE)
   - Connect client
   ↓
6. discoverTools()
   - Call listTools() on each client
   - Cache all tools with qualified names
   ↓
7. server.connect(transport)
   - Start stdio transport
   - Ready to receive requests
```

### get_tool_definitions Flow

```
1. Parent client calls get_tool_definitions
   ↓
2. server.ts receives tools/call request
   ↓
3. clientManager.getTools()
   - Returns cached tool list
   ↓
4. generateTypeDefinitions(tools)
   - Convert each tool's JSON Schema to TypeScript
   - Generate interfaces and function declarations
   ↓
5. Return TypeScript code as text
```

### execute_toolcall_script Flow

```
1. Parent client calls execute_toolcall_script with code
   ↓
2. server.ts receives tools/call request
   ↓
3. Validate code and timeout parameters
   ↓
4. executeSandbox(code, clientManager, config)
   ↓
5. Create sandbox context with:
   - Console methods (capturing output)
   - __callTool injected function
   - Basic JavaScript globals
   ↓
6. generateRuntimeStubs(tools)
   - Create function stubs that call __callTool
   ↓
7. Wrap code: stubs + async IIFE wrapper + user code
   ↓
8. vm.Script.runInContext()
   - Execute with timeout
   ↓
9. User code calls tool functions:
   toolFunction() → __callTool() → clientManager.callTool() → child server
   ↓
10. Collect result, logs, errors
    ↓
11. formatSandboxResult()
    ↓
12. Return formatted text
```

### Tool Call Flow (Internal)

```
1. User code: await filesystem_read_file({ path: "/tmp/file.txt" })
   ↓
2. Stub function: __callTool("filesystem.read_file", { path: ... })
   ↓
3. clientManager.callTool(qualifiedName, args)
   ↓
4. Parse "filesystem.read_file" → server="filesystem", tool="read_file"
   ↓
5. Get client for "filesystem" server
   ↓
6. client.callTool({ name: "read_file", arguments: { path: ... } })
   ↓
7. Child server processes request
   ↓
8. Return ToolResult to sandbox
   ↓
9. User code receives result
```

## Building and Running

### Development Commands

```bash
# Setup and Build
bun install              # Install dependencies (Bun preferred for performance)
bun run build            # Compile TypeScript to dist/

# Development
bun run dev              # Watch mode (rebuild on changes)
bun dist/index.js --config mcp.json  # Run server manually

# Testing
bun test                 # All tests (unit + integration)
bun run test:unit        # Unit tests only (fast)
bun run test:integration # Integration tests (requires Python server)
bun run test:docker      # Docker-based isolated tests

# Code Quality
bun run lint             # Check linting
bun run lint:fix         # Auto-fix linting issues
bun run format           # Format code with Prettier
bun run typecheck        # TypeScript type checking

# Testing Individual Components
bun tests/unit/run-unit-tests.ts              # Run unit tests directly
bun tests/integration/run-tests.ts            # Run integration tests directly
cd tests/mock-server && python pizza_shop.py  # Start test server standalone
```

**Note:** This project uses Bun for better performance, but npm/node also work. Integration tests require Python 3 with `uv` installed (uv auto-manages dependencies).

### Build Process

1. TypeScript compiler (tsc) runs
2. Reads tsconfig.json configuration
3. Compiles src/*.ts → dist/*.js
4. Generates source maps for debugging
5. Preserves ES modules format

### TypeScript Configuration

**tsconfig.json highlights:**

- Target: ES2022
- Module: ES2022 (native ES modules)
- Strict mode enabled
- Output to dist/ directory
- Source maps enabled
- Declaration files generated

## Dependencies

### Production Dependencies

**@modelcontextprotocol/sdk** (^1.20.0)

- Official MCP TypeScript SDK
- Provides Client and Server classes
- Includes transport implementations (stdio, SSE)
- Handles MCP protocol serialization

**zod** (^3.24.1)

- Runtime type validation
- Not heavily used currently but available for validation
- Could be used for more robust input validation

### Development Dependencies

**typescript** (^5.7.2)

- TypeScript compiler
- Type checking and code generation

**@types/node** (^22.10.5)

- TypeScript type definitions for Node.js APIs
- Required for vm, fs, path, etc.

## Test Infrastructure

### Pizza Shop Test Server (tests/mock-server/pizza_shop.py)
- FastMCP 2.0 Python server with 15 diverse tools
- Demonstrates simple calls, complex types, and chainable workflows
- Used by integration tests to validate full system behavior
- Uses modern Python tooling: `pyproject.toml` with `uv`
- Run with: `uv run pizza_shop.py` (auto-installs dependencies in venv)

### VM Test Programs (tests/vm-tests/)
- 7 TypeScript test programs demonstrating different patterns:
  - Simple tool calls
  - Complex types (arrays, objects, JSON)
  - Chaining (create_pizza → create_order → update_status)
  - Error handling (try/catch)
  - Parallel calls (Promise.all)
  - Control flow (loops, conditionals)
  - Data transformation (map/filter/reduce)

### Integration Tests (tests/integration/run-tests.ts)
- Starts Pizza Shop server
- Starts mcpcodeserver connected to Pizza Shop
- Validates 15 tools discovered
- Executes all 7 VM test programs
- Verifies results match expected output

## Testing Strategy

### Unit Tests

- `jsonSchemaToTypeScript()` - Schema conversion correctness
- `generateTypeDefinitions()` - TypeScript output validation
- `generateRuntimeStubs()` - Function stub generation
- `validateCode()` - Syntax validation

### Integration Tests

- MCPClientManager connection to mock servers
- Tool discovery and caching
- Tool call routing
- Error handling

### End-to-End Tests

- Full server startup with test config
- get_tool_definitions returns valid TypeScript
- execute_toolcall_script executes code correctly
- Sandbox security boundaries
- Timeout handling

## Extension Points

### Adding New Features

#### 1. Tool Filtering

Add ability to filter which tools are exposed:

- Modify `MCPConfig` to include filter patterns
- Filter in `discoverTools()` before caching
- Useful for security or reducing complexity

#### 2. Dynamic Server Management

Add tools to enable/disable servers at runtime:

- Add `enable_server` and `disable_server` meta-tools
- Modify client-manager to support dynamic add/remove
- Refresh tool definitions after changes

#### 3. Persistent Sandbox State

Allow state to persist between executions:

- Add session management to sandbox
- Store variables in per-session context
- Clear on explicit reset or timeout

#### 4. Enhanced Security

Improve sandbox isolation:

- Use isolated-vm instead of vm module
- Add resource limits (memory, CPU)
- Implement allowlist for dangerous operations

#### 5. TypeScript Type Checking

Add real TypeScript type checking before execution:

- Integrate ts-morph or TypeScript compiler API
- Check user code against generated definitions
- Return type errors before execution

#### 6. Tool Result Caching

Cache tool results to avoid redundant calls:

- Add cache layer in client-manager
- Configurable TTL per tool or server
- Cache invalidation strategies

## Common Development Tasks

### Adding New Sandbox Capabilities
1. Modify `sandbox` object in `src/sandbox.ts`
2. Update security documentation in README.md
3. Add unit tests in `tests/unit/sandbox.test.ts`

### Adding Tool Filtering Features
1. Update `MCPConfig` type in `src/types.ts`
2. Modify `MCPClientManager.discoverTools()` in `src/client-manager.ts`
3. Add configuration examples to README.md

### Debugging Connection Issues
```bash
# Test child server independently
python3 tests/mock-server/pizza_shop.py

# Test CLI parsing
node dist/index.js --help
node dist/index.js --config mcp.json.example

# Add debug logging in client-manager.ts
console.error('[client-manager]', ...);
```

### Debugging VM Execution Issues
- Add logging in `src/sandbox.ts` with `console.error('[sandbox]', ...)`
- Check timeout settings (default 30s, max 5min)
- Verify tool names are fully qualified (`serverName.toolName`)

## Design Principles for AI Agents

### 🚨 CRITICAL: Never Hardcode Test-Specific Logic

**NEVER** hardcode knowledge about specific test servers or their tools in the main implementation. This is a major anti-pattern that breaks the generic nature of the proxy.

#### ❌ WRONG - Don't Do This:
```typescript
// BAD: Hardcoded knowledge about test server
if (promptName.includes("pizza") || promptName.includes("recommendation")) {
  toolExamples = `
   - \`${serverName}.list_available_toppings()\` to see available toppings
   - \`${serverName}.get_size_info()\` to see pizza sizes
   - \`${serverName}.calculate_pizza_price()\` to calculate prices`;
}
```

#### ✅ CORRECT - Do This Instead:
```typescript
// GOOD: Dynamic discovery of actual tools
const tools = this.getTools().filter(tool => tool.serverName === serverName);
const toolNames = tools.slice(0, 5).map(tool => tool.name);
toolExamples = toolNames.map(name => `   - \`${serverName}.${name}()\``).join("\n");
```

### Generic Implementation Requirements

1. **Dynamic Discovery**: Always discover tools/resources/prompts dynamically from child servers
2. **No Test Dependencies**: Never reference `pizzashop`, `filesystem`, or any specific test server in main code
3. **Server-Agnostic**: All logic should work with any MCP server, not just our test servers
4. **Configuration-Driven**: Use configuration and runtime discovery, not hardcoded assumptions

### Test Server Isolation

- **Test servers** (`tests/mock-server/`) are for testing only
- **Main implementation** (`src/`) must be completely generic
- **Integration tests** verify the generic implementation works with test servers
- **Never** let test server specifics leak into production code

## Key Design Decisions

**Why VM sandbox?** Balances isolation and functionality. Alternative: isolated-vm for stronger security.

**Why TypeScript generation?** LLMs excel at code generation. Provides familiar syntax, type safety, and enables IDE autocomplete.

**Why qualified names?** `serverName.toolName` prevents conflicts when multiple servers expose tools with the same name.

**Why Bun?** Faster startup, native TypeScript support, better performance for dev/test cycles. npm/node still supported.

## Important Notes

- **ESM modules:** All imports must use `.js` extensions (e.g., `import { foo } from './bar.js'`) even when importing `.ts` files
- **Strict TypeScript:** Project uses strict mode - all functions should have explicit return types
- **Error context:** Wrap errors with context (server name, tool name) for better debugging
- **Sandbox security:** The VM context is NOT a security boundary - only execute trusted code
- **Test dependencies:** Integration tests require Python 3 with `uv` installed (dependencies auto-managed)

## Troubleshooting

### Common Issues

#### "Config file not found"

- Check config path is correct
- Use absolute path or relative to CWD
- Default is ./mcp.json

#### "Command is required for stdio transport"

- Each stdio server needs a command field
- Check mcp.json syntax
- Example: `"command": "npx"`

#### "Failed to discover tools from [server]"

- Check child server is working independently
- Verify command/args are correct
- Check environment variables are set
- Look for stderr output from child process

#### "Tool not found: [toolName]"

- Use fully qualified name: `serverName.toolName`
- Check tool discovery logs on startup
- Use get_tool_definitions to see available tools

#### "Execution timeout"

- Code took longer than timeout (default 30s)
- Increase timeout parameter (max 5min)
- Check for infinite loops
- Look for slow tool calls

#### TypeScript compilation errors

- Run `npm run build` to see detailed errors
- Check all imports use .js extensions (ESM requirement)
- Verify types match between modules
- Update dependencies if needed

## Performance Considerations

### Optimization Opportunities

1. **Tool Discovery Caching**
   - Currently rediscovers tools on every startup
   - Could cache to disk and refresh periodically
   - Trade-off: stale tool definitions vs startup speed

2. **Parallel Tool Calls**
   - User code can use Promise.all() for parallel calls
   - Client manager already supports concurrent calls
   - No additional work needed

3. **Code Compilation Caching**
   - VM.Script compilation could be cached
   - Hash code and reuse compiled scripts
   - Significant speedup for repeated code

4. **Transport Selection**
   - stdio has process spawn overhead
   - SSE may be faster for frequent calls
   - Consider using SSE for high-traffic servers

## Security Considerations

### Current Security Posture

**Sandbox Limitations:**

- Node.js VM is not a security boundary
- Can be escaped by determined attacker
- Suitable for trusted code only

**Recommendations:**

- Only execute code from trusted sources
- Consider this a convenience feature, not a security feature
- For untrusted code, use isolated-vm or external sandboxing

**Environment Variables:**

- Child servers receive parent process env
- May leak sensitive information
- Use explicit env override to prevent

**Network Access:**

- Child servers may have network access
- No restrictions on what tools can do
- Trust child servers as much as parent

## Contributing Guidelines

### Code Style

- Use TypeScript strict mode
- Document all public functions with JSDoc
- Use explicit return types
- Prefer async/await over promises
- Use const for immutable values
- Use meaningful variable names

### Adding New Modules

1. Create in src/ directory
2. Add to types.ts if defining shared types
3. Use .js extensions in imports (ESM requirement)
4. Export public API clearly
5. Document with comments

### Error Handling

- Use try/catch for async operations
- Wrap errors with context
- Log errors to stderr (console.error)
- Return user-friendly error messages
- Include stack traces for debugging

## Future Roadmap

Potential features for future versions:

1. **WebAssembly Sandbox** - More secure execution environment
2. **Tool Composition** - Define new tools as compositions of existing tools
3. **Streaming Results** - Stream console output and partial results
4. **TypeScript Language Server** - Real-time type checking and autocomplete
5. **Debugging Support** - Step through code execution
6. **Resource Limits** - Memory and CPU constraints
7. **Multi-Language Support** - Python, JavaScript, etc.
8. **Tool Marketplace** - Discover and install MCP servers
9. **Observability** - Metrics, logging, tracing
10. **Testing Framework** - Built-in test runner for user code

## CI/CD (GitHub Actions)

**.github/workflows/ci.yml** - Runs on push/PR
- Linting, formatting, type checking
- Unit and integration tests (with uv for Python dependencies)
- Build matrix: Ubuntu/macOS/Windows × Node 20/22/23

**.github/workflows/release.yml** - Runs on version tags (v*)
- Creates GitHub release with automated notes

## Testing with Claude Code

To quickly test mcpcodeserver with Claude Code using the Pizza Shop test server:

```bash
# One-command setup
./setup-claude-code-test.sh
```

This script will:
1. Install dependencies
2. Build the project
3. Install Python test server dependencies
4. Show you the exact configuration to add to Claude Code

After running the script, add the displayed configuration to your Claude Code MCP config file:
- **macOS/Linux:** `~/.config/claude-code/mcp_config.json`
- **Windows:** `%APPDATA%\claude-code\mcp_config.json`

Then restart Claude Code and try commands like:
- "Use get_tool_definitions to show me what tools are available"
- "Use execute_toolcall_script to get the store hours"
- "Use execute_toolcall_script to create a pizza order with pepperoni"

See **TESTING_WITH_CLAUDE.md** for detailed instructions and debugging tips.

## Resources

- [MCP Specification](https://modelcontextprotocol.io)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [FastMCP Documentation](https://github.com/jlowin/fastmcp)
- [Node.js VM Documentation](https://nodejs.org/api/vm.html)
- [JSON Schema](https://json-schema.org/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---
> Source: [zbowling/mcpcodeserver](https://github.com/zbowling/mcpcodeserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
