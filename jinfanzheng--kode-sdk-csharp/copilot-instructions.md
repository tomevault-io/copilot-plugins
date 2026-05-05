## kode-sdk-csharp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Kode Agent SDK for C# is a sophisticated, event-driven AI Agent runtime built on .NET 10. It provides a comprehensive framework for building AI applications with tool execution, state persistence, multi-agent collaboration, and MCP (Model Context Protocol) integration.

### Key Features
- **Multi-model Support**: Anthropic Claude and OpenAI GPT models
- **Event-Driven Architecture**: Three-channel event system (Progress, Control, Monitor)
- **Tool System**: 20+ built-in tools with custom tool development support
- **State Persistence**: JSON file storage and Redis distributed storage
- **Permission Control**: Fine-grained tool permission management
- **Skills System**: Progressive skill discovery and activation
- **Sub-Agent Delegation**: Task delegation to specialized sub-agents
- **MCP Integration**: Native Model Context Protocol support
- **Source Generator**: Compile-time tool schema generation

## Build, Test, and Development Commands

### Prerequisites
- .NET 10.0 (specified in global.json)
- API keys for Anthropic or OpenAI (for examples that use models)

### Build Commands
```bash
# Build all projects
dotnet build

# Build with release configuration
dotnet build --configuration Release

# Build specific project
dotnet build src/Kode.Agent.Sdk/Kode.Agent.Sdk.csproj

# Clean build artifacts
dotnet clean
```

### Test Commands
```bash
# Run all tests
dotnet test

# Run tests with specific filter
dotnet test --filter "TestCategory=Unit"

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run tests with verbose output
dotnet test --logger "console;verbosity=detailed"
```

### Example Project Commands
```bash
# Run console examples (requires API keys)
cd examples/Kode.Agent.Examples
cp .env.example .env  # Edit with your API keys
dotnet run

# Run WebAPI example (OpenAI SSE compatible)
cd examples/Kode.Agent.WebApiAssistant
cp .env.example .env  # Edit with your API keys
dotnet run
```

### Package Management
```bash
# Restore all packages
dotnet restore

# Central package management (using Directory.Packages.props)
dotnet restore --force-evaluate
```

## High-Level Architecture

### Core Components

#### 1. Agent Core (`Kode.Agent.Sdk/Core/`)
- **Agent.cs**: Main agent implementation with event-driven architecture
- **EventBus.cs**: Three-channel event system (Progress, Control, Monitor)
- **MessageQueue.cs**: Message processing and queueing
- **BreakpointManager.cs**: State persistence and recovery
- **PermissionManager.cs**: Tool permission and approval control

#### 2. Infrastructure (`Kode.Agent.Sdk/Infrastructure/`)
- **Providers/**: Model providers (AnthropicProvider, OpenAIProvider)
- **Sandbox/**: Command execution environment (LocalSandbox)

#### 3. Tools System (`Kode.Agent.Sdk/Tools/`)
- **ToolRegistry.cs**: Tool registration and discovery
- **ToolBase.cs**: Base class for tool development
- **ToolAttributes.cs**: Tool metadata and attributes

#### 4. Storage (`Kode.Agent.Store.*`)
- **JsonAgentStore**: File-based storage (./.kode/agent-id/)
- **RedisAgentStore**: Redis distributed storage

#### 5. Source Generator (`Kode.Agent.SourceGenerator/`)
- Compiles tool schemas at build time (no reflection overhead)

#### 6. MCP Integration (`Kode.Agent.Mcp/`)
- **McpClientManager.cs**: MCP protocol client
- **McpToolProvider.cs**: Tool provider for MCP servers

### Key Patterns and Conventions

#### 1. Event-Driven Architecture
```csharp
// Three-channel event system
await foreach (var envelope in agent.EventBus.SubscribeAsync(EventChannel.Progress))
{
    switch (envelope.Event)
    {
        case TextChunkEvent textChunk:
            Console.Write(textChunk.Delta);  // Streaming output
            break;
        case ToolStartEvent toolStart:
            Console.WriteLine($"[tool] {toolStart.Call.Name} starting...");
            break;
        case DoneEvent done:
            Console.WriteLine("Conversation complete");
            break;
    }
}
```

#### 2. Tool Development with Source Generator
```csharp
[Tool("database_query")]
[Description("Execute SQL query")]
public partial class DatabaseQueryTool : ITool
{
    [ToolParameter("query", required: true)]
    public string Query { get; set; } = "";

    public async Task<ToolResult> ExecuteAsync(ToolContext context)
    {
        // Tool implementation
        return ToolResult.Success(result);
    }
}
```

#### 3. Dependency Injection
```csharp
// Services registration
services.AddKodeAgent(options =>
{
    options.DefaultModel = "claude-sonnet-4-20250514";
    options.StoreDirectory = "./.kode";
});

services.AddAnthropicProvider(options =>
{
    options.ApiKey = Configuration["Anthropic:ApiKey"]!;
});
```

#### 4. State Persistence
```csharp
// Agent automatically persists state during execution
await agent.RunAsync("执行任务");

// Create snapshot for safe branching point
var snapshotId = await agent.SnapshotAsync("backup");

// Resume from store
var restoredAgent = await Agent.ResumeFromStoreAsync("agent-id", deps);
```

## Important Configuration Files

### 1. `global.json`
- Defines .NET 10.0 SDK requirement
- Roll forward to latest feature version

### 2. `Directory.Build.props`
- Central build properties
- Target: net10.0
- LangVersion: preview
- TreatWarningsAsErrors: true

### 3. `Directory.Packages.props`
- Central package management
- Key dependencies: Microsoft.Extensions.*, Anthropic, OpenAI, StackExchange.Redis, ModelContextProtocol, Serilog

### 4. Solution Structure
```
Kode.Agent.sln
├── src/
│   ├── Kode.Agent.Sdk/          # Core SDK
│   ├── Kode.Agent.SourceGenerator/ # Source Generator
│   ├── Kode.Agent.Mcp/          # MCP Integration
│   ├── Kode.Agent.Store.Json/   # JSON Storage
│   ├── Kode.Agent.Store.Redis/  # Redis Storage
│   └── Kode.Agent.Tools.Builtin/# Built-in Tools
├── examples/
│   ├── Kode.Agent.Examples/     # Console Examples
│   └── Kode.Agent.WebApiAssistant/# WebAPI Example (OpenAI SSE compatible)
└── tests/
    └── Kode.Agent.Tests/        # Unit & Integration Tests
```

## Key Configuration Options

### Agent Configuration
```csharp
var config = new AgentConfig
{
    Model = "claude-sonnet-4-202514",
    SystemPrompt = "You are a helpful assistant.",
    MaxIterations = 20,
    Tools = ["fs_read", "fs_write", "bash_run"],
    Permissions = new PermissionConfig
    {
        Mode = "auto",                       // auto | approval | readonly
        RequireApprovalTools = ["bash_run"], // Tools requiring approval
        DenyTools = ["fs_rm"]                // Forbidden tools
    },
    SandboxOptions = new SandboxOptions
    {
        WorkDir = "./workspace",
        EnforceBoundary = true,
        AllowedPaths = ["/allowed/path"]
    }
};
```

### Permission Modes
- `auto`: Default allow (can be refined with allow/deny/require lists)
- `approval`: All tools require manual approval
- `readonly`: Based on tool metadata, denies tools that mutate state
- Custom: Host-registered permission handlers

### MCP Configuration
MCP servers are configured in `appsettings.json` under `McpServers` section:

```json
{
  "McpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--headless=true"]
    },
    "glm-web-search": {
      "transport": "streamableHttp",
      "url": "https://open.bigmodel.cn/api/mcp/web_search_prime/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

Supported transports: `stdio`, `http`, `streamableHttp`, `sse`

### Logging Configuration (Serilog)
The WebAPI example uses Serilog with:
- Console output with custom template
- File output to `logs/kode-.log` with daily rolling
- 7-day log retention
- Microsoft framework logs set to Warning level

## MCP Integration Details

### MCP Tools Naming Convention
MCP tools are registered with namespaced names: `mcp__{serverName}__{toolName}`
- Example: `mcp__chrome-devtools__take_screenshot`

### MCP Allowlist Handling
Due to namespaced MCP tool names, use `*` in `AllowTools` to permit all MCP tools:
```json
{
  "Kode": {
    "AllowTools": "*,fs_read,fs_write,..."
  }
}
```

Alternatively, the application can automatically merge MCP tool names into the allowlist at runtime (similar to TypeScript implementation).

## Development Workflow

### 1. Setting Up Development Environment
```bash
# Clone and install dependencies
git clone <repository>
cd kode-sdk-csharp
dotnet restore

# Verify build
dotnet build

# Run tests to ensure everything works
dotnet test
```

### 2. Creating a Custom Tool
1. Use the Source Generator pattern for type safety
2. Implement `ITool` interface
3. Use `[Tool]` and `[ToolParameter]` attributes
4. Handle cancellation via `ToolContext.CancellationToken`

### 3. Testing Best Practices
- Unit tests for tool logic
- Integration tests for agent workflows
- Use Moq for dependency mocking
- Use FluentAssertions for assertions

### 4. Common Pitfalls
- **Tool Registration**: Ensure tools are registered before agent creation
- **Cancellation**: Always pass CancellationToken to async operations
- **State Persistence**: Agent saves state automatically, no manual calls needed
- **Memory Management**: Use `await using` for Agent disposal
- **MCP Tools**: Remember MCP tools use namespaced naming (`mcp__*`)
- **Permission Wildcard**: Use `*` in AllowTools to permit all tools including MCP tools

## Advanced Features

### Skills System
Progressive skill discovery and activation:
```csharp
// Configure skills paths
var skillsConfig = new SkillsConfig
{
    Paths = ["./.kode/skills", "./skills"],
    Include = ["code-review", "testing"],
    ValidateOnLoad = true
};

// Activate skills on demand
var skill = await skillsManager.ActivateAsync("code-review");
```

### Sub-Agent Delegation
Delegate tasks to specialized agents:
```csharp
var templates = new List<AgentTemplate>
{
    new AgentTemplate
    {
        Id = "code-analyst",
        System = "You are a code analyst",
        Tools = ["fs_read", "fs_grep"]
    }
};
```

### Hooks and Interception
Intercept agent lifecycle events:
```csharp
agent.On("agent_created", context =>
{
    // Custom logic on agent creation
    return Task.CompletedTask;
});
```

## Performance Considerations

1. **Tool Concurrency**: Configure `MaxToolConcurrency` in AgentConfig
2. **Memory Management**: Properly dispose agents with `await using`
3. **State Storage**: Use Redis for distributed deployments
4. **CancellationToken**: Always respect cancellation tokens
5. **Source Generator**: Prefer over reflection for better performance

## Troubleshooting

### Common Issues
1. **Model API Errors**: Check API key and network connectivity
2. **Tool Registration**: Verify tools are registered before agent creation
3. **State Corruption**: Delete `.kode/agent-id` directory to reset state
4. **Permission Denied**: Review permission configuration and tool metadata
5. **MCP Connection Failed**: Verify URL, headers, and transport type for HTTP-based MCP servers
6. **MCP Tools Not Available**: Ensure `*` is in `AllowTools` or specific `mcp__*` patterns are added

### Debugging
- Enable Serilog structured logging (already configured in WebAPI example)
- Subscribe to Monitor channel for error events
- Use breakpoint states to inspect agent execution flow
- Check `logs/kode-.log` for detailed application logs

## Resources

- Documentation: `docs/ADVANCED_GUIDE.md` and `docs/API_REFERENCE.md`
- TypeScript Alignment: `docs/TS_ALIGNMENT.md` for cross-language compatibility
- Examples: `examples/Kode.Agent.Examples/` and `examples/Kode.Agent.WebApiAssistant/`
- Tests: `tests/Kode.Agent.Tests/` for usage patterns

---
> Source: [JinFanZheng/kode-sdk-csharp](https://github.com/JinFanZheng/kode-sdk-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
