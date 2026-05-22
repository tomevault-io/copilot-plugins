## tools-implementation

> - MUST: Register tools using `register_tools` function in `src/mcp/tools.rs`

# tools-implementation

### Tool Registration Pattern
- MUST: Register tools using `register_tools` function in `src/mcp/tools.rs`
- MUST: Include schema and description for each tool
- AVOID: Direct function calls without registration
- WHY: Enables dynamic tool discovery and validation
- EXAMPLE: 
```rust
// src/mcp/tools.rs
register_tools(router, vec![
    ("file_edit", file_edit_schema(), "Edit file content"),
    ("grep_search", grep_search_schema(), "Search files")
])
```
Importance: 95

### Git Integration Flow
- MUST: Use optional Git commits in file operations via `file_edit`, `create_directory`, `move_or_rename`
- MUST: Include commit message for each Git operation
- AVOID: Direct Git commands without validation
- WHY: Maintains version control integration with file operations
- EXAMPLE: `src/mcp/tools.rs:file_edit` handling Git commits
Importance: 85

### Path Validation
- MUST: Validate all paths using `validate_path_or_error` from `src/mcp/tools.rs`
- MUST: Check against allowed directories list
- AVOID: Direct file system access without validation
- WHY: Ensures security by restricting operations to allowed paths
- EXAMPLE: `validate_paths_or_error` for move operations
Importance: 90

### Tool Implementation Structure
- MUST: Implement each tool with standardized error handling
- MUST: Return structured responses with status and data
- AVOID: Mixed responsibility tools combining multiple operations
- WHY: Maintains consistent tool behavior and error handling
- EXAMPLE: 
```rust
// src/mcp/tools.rs
fn grep_search(args: Value) -> Result<Value, Error> {
    // Standardized implementation
}
```
Importance: 80

### Grep Tool Configuration
- MUST: Support recursive and case-sensitive options in `grep_search`
- MUST: Validate search paths before execution
- AVOID: Hardcoded grep options
- WHY: Provides flexible search capabilities with validation
- EXAMPLE: `src/mcp/tools.rs:grep_search` implementation
Importance: 75

### Time Tools Implementation
- MUST: Use `current_time` and `get_local_time` for time operations
- MUST: Support city-specific time queries
- AVOID: Direct system time access
- WHY: Standardizes time-related operations across the system
- EXAMPLE: `src/mcp/tools.rs:current_time` implementation
Importance: 70

$END$

---
> Source: [TexasFortress-AI/rs_filesystem](https://github.com/TexasFortress-AI/rs_filesystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
