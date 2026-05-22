## rs-filesystem

> - Only modify code directly relevant to the specific request. Avoid changing unrelated functionality.


START SPECIFICATION:
# main-overview

## Development Guidelines

- Only modify code directly relevant to the specific request. Avoid changing unrelated functionality.
- Never replace code with placeholders like `# ... rest of the processing ...`. Always include complete code.
- Break problems into smaller steps. Think through each step separately before implementing.
- Always provide a complete PLAN with REASONING based on evidence from code and logs before making changes.
- Explain your OBSERVATIONS clearly, then provide REASONING to identify the exact issue. Add console logs when needed to gather more information.


### Core Business Architecture

The project implements a Modular Command Protocol (MCP) server that provides file system operations and tools through a JSON-RPC interface. Key components:

1. Tool Management System (`src/mcp/tools.rs`)
- Central registry for system tools with Git integration
- File editing capabilities with version control
- Directory operations with commit tracking
- Custom search functionality
- Time-based operations for different cities

2. Access Control Layer (`src/mcp/utilities.rs`)
- Path validation system for allowed directories
- Canonicalization of paths for security
- Server capability declarations

3. Resource Management (`src/mcp/resources.rs`)
- Directory access control through allowed paths
- Resource listing and retrieval system

4. Prompt System (`src/mcp/prompts.rs`)
- Dynamic prompt generation based on operations
- Argument-based content customization
- Operation-specific metadata handling

### Code Conventions

#### JSON-RPC Implementation
- MUST: Use `rpc_router` crate for endpoint routing
- AVOID: Direct JSON manipulation without RPC protocol
- WHY: Ensures consistent MCP protocol compliance
- EXAMPLE: `src/main.rs:build_rpc_router()`

#### Git Integration
- MUST: Include commit messages for all file operations
- AVOID: Direct file system changes without version control
- WHY: Maintains audit trail of system changes
- EXAMPLE: `src/mcp/tools.rs:file_edit()`

#### Path Validation
- MUST: Use `validate_path_or_error` for all file operations
- AVOID: Direct path manipulation without validation
- WHY: Security and access control requirements
- EXAMPLE: `src/mcp/utilities.rs:validate_path_or_error()`

#### Tool Registration
- MUST: Register tools through central registry
- AVOID: Standalone tool implementations
- WHY: Ensures discoverability and consistent interface
- EXAMPLE: `src/mcp/tools.rs:register_tools()`

Importance Scores:
- Tool Management: 95 (Core business functionality)
- Access Control: 90 (Critical security component)
- Resource Management: 85 (Key infrastructure)
- Prompt System: 80 (Essential user interaction)

$END$
END SPECIFICATION

---
> Source: [TexasFortress-AI/rs_filesystem](https://github.com/TexasFortress-AI/rs_filesystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
