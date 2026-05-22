## path-validation

> - MUST: Use `validate_path_or_error` from `src/mcp/utilities.rs` for all file operations

# path-validation

### Secure Path Validation
- MUST: Use `validate_path_or_error` from `src/mcp/utilities.rs` for all file operations
- MUST: Implement path canonicalization before validation using `std::fs::canonicalize`
- AVOID: Manual string concatenation or direct path comparison
- WHY: Prevents directory traversal attacks and ensures operations stay within allowed boundaries
- EXAMPLE: 
```rust
// src/mcp/tools.rs
validate_path_or_error(&file_path)?;
```

### Multi-Path Operations
- MUST: Use `validate_paths_or_error` from `src/mcp/utilities.rs` for move/rename operations
- MUST: Validate both source and target paths before any file operation
- AVOID: Performing operations without checking both paths
- WHY: Ensures atomic validation for operations involving multiple paths
- EXAMPLE:
```rust
// src/mcp/tools.rs
validate_paths_or_error(&source_path, &target_path)?;
```

### Allowed Directories Registry
- MUST: Use `is_path_allowed` from `src/mcp/utilities.rs` to check against allowed directories
- MUST: Retrieve allowed directories through `resources/allowed_directories` endpoint
- AVOID: Hardcoding directory paths or bypass validation
- WHY: Centralizes directory access control and maintains security boundary
- EXAMPLE:
```rust
// src/mcp/resources.rs
resource_read("Allowed Directories")
```

### Path Validation Testing
- MUST: Test path validation using `src/mcp/tools_test.rs`
- MUST: Include test cases for:
  - Valid paths within allowed directories
  - Invalid paths outside allowed directories
  - Path canonicalization edge cases
- AVOID: Testing without canonicalized paths
- WHY: Ensures robust validation across different path formats and scenarios

### Git Integration Security
- MUST: Validate paths before Git operations in `file_edit` function
- MUST: Check paths before creating Git commits
- AVOID: Direct Git operations without path validation
- WHY: Maintains security boundary for version-controlled files
- EXAMPLE:
```rust
// src/mcp/tools.rs
validate_path_or_error(&file_path)?;
// Only then proceed with Git commit
```

Importance Scores:
- Secure Path Validation: 95 (core security mechanism)
- Multi-Path Operations: 90 (critical for file operations)
- Allowed Directories Registry: 85 (key security boundary)
- Path Validation Testing: 80 (ensures security compliance)
- Git Integration Security: 75 (maintains version control security)

$END$

---
> Source: [TexasFortress-AI/rs_filesystem](https://github.com/TexasFortress-AI/rs_filesystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
