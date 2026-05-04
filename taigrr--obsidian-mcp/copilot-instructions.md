## obsidian-mcp

> This file provides guidance to AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

obsidian-mcp is a Model Context Protocol (MCP) server implemented in Go that provides a universal AI bridge for Obsidian vaults. It enables any MCP-compatible AI assistant (Claude, ChatGPT, etc.) to safely read and write notes in Obsidian vaults while preserving YAML frontmatter and enforcing security boundaries.

## Development Commands

### Build and Test

```bash
go build ./...                    # Build all packages
go build -o obsidian-mcp ./cmd/obsidian-mcp  # Build the binary
go test ./...                     # Run all tests
go test -v ./internal/filesystem  # Run specific package tests with verbose output
go test -run TestService_Patch ./internal/filesystem  # Run specific test
```

### Running Locally

```bash
go run ./cmd/obsidian-mcp /path/to/vault  # Run directly
./obsidian-mcp /path/to/vault             # Run compiled binary
```

### Code Quality

```bash
goimports -w .   # Format code
go vet ./...     # Check for issues
```

### Build with Version

```bash
go build -ldflags "-X main.Version=1.0.0" -o obsidian-mcp ./cmd/obsidian-mcp
```

## Architecture

### Project Structure

```
cmd/obsidian-mcp/     # Main application entry point
internal/
  filesystem/         # FileSystemService - all file operations
  frontmatter/        # FrontmatterHandler - YAML parsing/validation
  pathfilter/         # PathFilter - security filtering
  search/             # SearchService - content search
  types/              # Shared type definitions
  uri/                # Obsidian URI generation
```

### Core Components

**cmd/obsidian-mcp/main.go** - MCP server entry point

- Handles command-line arguments (--help, --version, vault path)
- Initializes all services
- Registers 12 MCP tools using github.com/modelcontextprotocol/go-sdk
- Uses generic AddTool function with typed input/output structs

**internal/filesystem/filesystem.go** - FileSystemService

- Orchestrates all file operations with security enforcement
- Path resolution and security validation (prevents path traversal)
- Implements: ReadNote, WriteNote, PatchNote, DeleteNote, MoveNote, ListDirectory
- Batch operations: ReadMultipleNotes, GetNotesInfo
- Tag management in frontmatter and inline tags

**internal/frontmatter/frontmatter.go** - FrontmatterHandler

- Uses gopkg.in/yaml.v3 for parsing/stringifying YAML frontmatter
- Validates frontmatter structure (blocks functions, symbols, invalid types)
- Gracefully handles notes without frontmatter
- Preserves original content with OriginalContent field

**internal/pathfilter/pathfilter.go** - PathFilter

- Security layer that filters allowed paths and file types
- Blocks: .obsidian/, .git/, node_modules/, system files
- Allows: .md, .markdown, .txt files
- Uses glob patterns converted to regex

**internal/search/search.go** - SearchService

- Content and frontmatter search with configurable case sensitivity
- Returns token-optimized results with minified field names (p, t, ex, mc, ln)
- Configurable search limits (max 20 results)

**internal/types/types.go** - Type definitions

- All shared structs for parameters, results, and data

### Key Design Patterns

**Service Layer Architecture**

- Each service has a single responsibility
- Services are dependency-injected in main.go
- Services can be tested independently

**Security-First Design**

- All paths validated through PathFilter before any operation
- Path traversal prevention via filepath.Rel check
- Confirmation required for destructive operations (delete requires confirmPath match)
- Read-only system directories automatically filtered

**Token Optimization**

- JSON responses use minified field names (e.g., fm instead of frontmatter)
- Search results use compact format: {p, t, ex, mc, ln}

**Error Handling**

- Comprehensive error messages with context
- Operations return structured results with Success boolean
- Failed batch operations return partial results (Successful + Failed arrays)

### MCP Tool Handler Pattern

Tools use the generic AddTool function with typed structs:

```go
type ReadNoteInput struct {
    Path        string `json:"path" jsonschema:"Path to the note relative to vault root"`
    PrettyPrint bool   `json:"prettyPrint,omitempty" jsonschema:"Format JSON response with indentation"`
}

type ReadNoteOutput struct {
    Frontmatter map[string]any `json:"fm"`
    Content     string         `json:"content"`
}

func handleReadNote(ctx context.Context, req *mcp.CallToolRequest, input ReadNoteInput) (*mcp.CallToolResult, ReadNoteOutput, error) {
    // Implementation
}

mcp.AddTool(server, &mcp.Tool{Name: "read_note", Description: "Read a note"}, handleReadNote)
```

### Write Modes

Three modes for writing notes:

1. **overwrite** (default) - Replace entire file
2. **append** - Add content to end, merge frontmatter
3. **prepend** - Add content to beginning, merge frontmatter

### Patch Operation

The patch_note tool efficiently updates parts of a note:

- Finds and replaces exact string matches (including whitespace/newlines)
- replaceAll: false (default) fails if multiple matches found
- replaceAll: true replaces all occurrences

## Testing

Tests use Go's standard testing package with table-driven tests:

```bash
go test ./...                          # Run all tests
go test -v ./internal/filesystem       # Verbose output for package
go test -run TestService_Patch ./...   # Run tests matching pattern
go test -cover ./...                   # Coverage report
```

When writing tests:

- Test both success and error cases
- Test path security (path traversal, access denied)
- Test frontmatter parsing edge cases
- Use t.TempDir() or os.MkdirTemp() for test vaults

## Security Considerations

When modifying file operations:

- Always validate paths through PathFilter
- Always use resolvePath() to prevent traversal
- Never expose system directories or configuration
- Validate frontmatter before writing (check for functions)
- Require confirmation for destructive operations

## Dependencies

- github.com/modelcontextprotocol/go-sdk v1.0.0 - Official MCP Go SDK
- gopkg.in/yaml.v3 - YAML parsing for frontmatter

---
> Source: [taigrr/obsidian-mcp](https://github.com/taigrr/obsidian-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
