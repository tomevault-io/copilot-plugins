## tron-foundry-proxy

> Update the README.md documentation to reflect the new Request/Response Processing and Override Rules that have been implemented in the codebase. The documentation needs to accurately describe:

# Augment Guidelines and Scratchpad

## Current Task: Update README.md Documentation

### Task Description:
Update the README.md documentation to reflect the new Request/Response Processing and Override Rules that have been implemented in the codebase. The documentation needs to accurately describe:
1. Current request/response processing logic
2. Override rules and when they apply
3. Clear examples of how features work
4. Maintain consistency with existing documentation style

### Implementation Analysis:
Based on examination of src/main.rs, the current implementation includes:

**Override Rules:**
1. **eth_getTransactionCount**: Returns "0x0" directly without forwarding
2. **eth_call**: Parameter normalization (input/data fields, chainId removal)
3. **Block Response Enhancement**: Fixes invalid/missing stateRoot in block responses

**Request/Response Processing:**
- JSON-RPC request parsing and validation
- Method-specific parameter normalization
- Response enhancement for block methods
- Proper JSON-RPC 2.0 compliance (no null error fields)
- Content-Length header correction when responses are modified
- Comprehensive logging at multiple levels

### Task Plan:
[X] Examine current implementation in main.rs
[X] Review existing README.md structure
[X] Update "Request Processing" section with detailed explanations
[X] Add examples for each override rule
[X] Document response processing enhancements
[X] Ensure documentation accuracy matches implementation

### Final Status: ✅ COMPLETED
- Updated README.md with comprehensive documentation of current features
- Added detailed explanations of all three override rules with examples
- Documented request/response processing flow
- Added JSON-RPC 2.0 compliance details
- Included header management and error handling information
- Enhanced Features section to reflect current capabilities

### Previous Task Status: ✅ COMPLETED - Fix cast block Deserialization Error
- Cast block command now works correctly for all block numbers
- Fixed invalid stateRoot handling and JSON-RPC response structure

### Analysis of Current Implementation:
- The proxy already has some logging (lines 214-218, 225-226)
- Response headers are being copied from destination to client (lines 231-238)
- The implementation looks correct at first glance
- Need to test destination API directly and add more detailed logging

### Previous Implementation Status:
[X] Initialize Rust project with Cargo.toml
[X] Add necessary dependencies (axum, tokio, serde, clap, reqwest, tracing)
[X] Create main.rs with CLI argument parsing
[X] Implement HTTP server with axum
[X] Create JSON-RPC request/response structures
[X] Implement proxy logic with request forwarding
[X] Add eth_getTransactionCount override
[X] Add eth_call parameter normalization
[X] Add block response enhancement (stateRoot)
[X] Add structured logging
[X] Fix compilation errors (header conversion)
[X] Test the implementation
[X] Create README documentation

### Technology Choices:
- HTTP Server: axum (modern, performant, good ecosystem)
- CLI: clap (standard for Rust CLI apps)
- HTTP Client: reqwest (standard for HTTP requests)
- JSON: serde_json (standard for JSON handling)
- Async Runtime: tokio (required by axum)
- Logging: tracing (structured logging)

### Lessons:
- When converting headers between reqwest and axum, use `axum::http::HeaderValue::from_bytes()` to convert reqwest header values to axum header values
- Remove unused imports (`Json`, `any`) to avoid warnings
- For GET request forwarding, need to handle URL construction with query parameters separately from POST requests
- JSON-RPC 2.0 specification: When creating successful responses, do NOT include `"error":null` field. Cast client expects either no error field or a proper error object, not null. Use `json!()` macro to create clean responses without null error fields.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xNebulaLumina)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/0xNebulaLumina)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
