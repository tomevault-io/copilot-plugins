## data-flow

> - MUST: Handle JSON-RPC requests through stdin/stdout flow using tokio::io

# Data-Flow

### Core Data Propagation
- MUST: Handle JSON-RPC requests through stdin/stdout flow using tokio::io
- AVOID: Direct file system access outside allowed directories
- WHY: Ensures controlled data access and consistent request handling
- EXAMPLE: `src/main.rs` - JSON-RPC router construction with tool registration
**Importance: 95**

### Tool Registration Flow
- MUST: Register tools with input schemas via `register_tools` in `src/mcp/tools.rs`
- AVOID: Global state for tool registration
- WHY: Maintains single source of truth for available tool definitions
- EXAMPLE: `src/mcp/tools.rs:register_tools()`
**Importance: 85**

### Resource Management Pipeline
- MUST: Flow resource requests through `resources_list` and `resource_read` in `src/mcp/resources.rs`
- AVOID: Direct filesystem access bypassing resource controllers
- WHY: Centralizes resource access control and validation
- EXAMPLE: `src/mcp/resources.rs:resources_list()`
**Importance: 80**

### Prompt Processing Chain
- MUST: Process prompts through `prompts_list` and `prompts_get` in `src/mcp/prompts.rs`
- AVOID: Mixing prompt and tool data flows
- WHY: Separates prompt handling from tool execution
- EXAMPLE: `src/mcp/prompts.rs:prompts_get()`
**Importance: 75**

### Path Validation Flow
- MUST: Route all path operations through validation in `src/mcp/utilities.rs`
- AVOID: Direct path manipulation without validation
- WHY: Ensures security by restricting operations to allowed directories
- EXAMPLE: `src/mcp/utilities.rs:validate_path_or_error()`
**Importance: 90**

### Git Integration Pipeline
- MUST: Handle Git operations through dedicated flows in file editing tools
- AVOID: Direct Git commands without error handling
- WHY: Maintains version control integration consistency
- EXAMPLE: `src/mcp/tools.rs:file_edit()`
**Importance: 70**

### Directory Operations Flow
- MUST: Route directory operations through validated tool handlers
- AVOID: Direct filesystem manipulation
- WHY: Ensures controlled access and proper error handling
- EXAMPLE: `src/mcp/tools.rs:create_directory()`
**Importance: 85**

$END$

---
> Source: [TexasFortress-AI/rs_filesystem](https://github.com/TexasFortress-AI/rs_filesystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
