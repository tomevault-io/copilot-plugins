## rpc-routing

> - MUST: Register RPC methods using `build_rpc_router` in `src/main.rs`

# rpc-routing

### JSON-RPC Router Registration
- MUST: Register RPC methods using `build_rpc_router` in `src/main.rs`
- AVOID: Direct handler function calls without router registration
- WHY: Ensures consistent routing and method discovery
- EXAMPLE:
```rust
// src/main.rs
router.append_route("tools/list", tools_list);
```
Importance: 95

### Method Registration Pattern
- MUST: Use the `tools_list` function in `src/mcp/tools.rs` to expose available tools
- AVOID: Hardcoding tool lists or bypassing registration
- WHY: Enables dynamic tool discovery and consistent schema validation
- EXAMPLE: 
```rust
// src/mcp/tools.rs
register_tools(router);
```
Importance: 90

### Handler Implementation
- MUST: Implement handlers returning proper JSON-RPC response types
- AVOID: Raw JSON returns or non-standard response formats
- WHY: Maintains protocol compliance and consistent error handling
- EXAMPLE:
```rust
// src/mcp/tools.rs
fn tools_list() -> JsonRpcResponse {
    // Handler implementation
}
```
Importance: 85

### Notification Processing 
- MUST: Process notifications using dedicated handlers in `src/main.rs`
- AVOID: Treating notifications as regular RPC calls
- WHY: Notifications require different handling than standard requests
- EXAMPLE:
```rust
// src/main.rs
notifications_initialized();
```
Importance: 80

### Resource Access Control
- MUST: Validate paths using `validate_path_or_error` in `src/mcp/utilities.rs`
- AVOID: Direct file system access without validation
- WHY: Enforces security boundaries for file operations
- EXAMPLE:
```rust
// src/mcp/utilities.rs
validate_path_or_error(path)?;
```
Importance: 85

### Tool Registration Workflow
- MUST: Register tools through the centralized registration system
- AVOID: Ad-hoc tool additions or direct router manipulation
- WHY: Maintains consistent tool discovery and documentation
- EXAMPLE: See `register_tools` in `src/mcp/tools.rs`
Importance: 75

$END$

---
> Source: [TexasFortress-AI/rs_filesystem](https://github.com/TexasFortress-AI/rs_filesystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
