## paperlessmcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
dotnet build                # Build the solution
dotnet test                 # Run all tests
dotnet test --filter "FullyQualifiedName~DocumentToolsTests"  # Run specific test class
dotnet test --logger "console;verbosity=detailed"             # Verbose test output
```

**Running locally:**
```bash
# Set required environment variables
export PAPERLESS_BASE_URL=https://your-instance.com
export PAPERLESS_API_TOKEN=your-token

# stdio mode (for Claude Desktop)
dotnet run --project PaperlessMCP -- --stdio

# HTTP mode (runs at http://localhost:5000/mcp)
dotnet run --project PaperlessMCP
```

## Architecture

PaperlessMCP is an MCP (Model Context Protocol) server that bridges Paperless-ngx with AI assistants. It supports dual transport modes: stdio for Claude Desktop integration and HTTP for remote usage.

### Key Components

- **Program.cs** - Entry point with dual transport setup (stdio vs HTTP). Configures DI, HttpClient with Polly retry policies, and MCP server registration.

- **Client/PaperlessClient.cs** - Central API client for all Paperless-ngx operations. Uses snake_case JSON serialization to match API conventions.

- **Client/PaperlessAuthHandler.cs** - DelegatingHandler that injects API token authentication.

- **Tools/** - MCP tool implementations, one file per entity type:
  - `DocumentTools.cs` - Search, CRUD, bulk operations, upload (base64 and file path)
  - `TagTools.cs`, `CorrespondentTools.cs`, `DocumentTypeTools.cs`, `StoragePathTools.cs` - Entity management with matching algorithms
  - `CustomFieldTools.cs` - Custom field definitions and assignments
  - `HealthTools.cs` - Ping and capabilities

- **Models/** - Request/response DTOs organized by entity (Documents/, Tags/, etc.). Uses `PaginatedResponse<T>` for list endpoints.

### Configuration

Environment variables (take precedence over appsettings.json):
- `PAPERLESS_BASE_URL` / `PAPERLESS_URL` - Paperless-ngx instance URL
- `PAPERLESS_API_TOKEN` / `PAPERLESS_TOKEN` - API authentication token
- `MAX_PAGE_SIZE` - Max items per page (default: 100)
- `MCP_PORT` - HTTP server port (default: 5000)

### Testing

Tests use xUnit with FluentAssertions for assertions, NSubstitute for mocking, and RichardSzalay.MockHttp for HTTP mocking. Test files mirror the main project structure under `PaperlessMCP.Tests/`.

## Conventions

- Use [Conventional Commits](https://www.conventionalcommits.org/) - version bumps are automatic based on commit type (`fix:` = patch, `feat:` = minor, `feat!:` = major)
- Trunk-based development: feature branches merge directly to `main`
- All destructive operations (delete, bulk operations) require explicit `confirm=true` and default to dry-run mode

## Versioning with Cocogitto (cog)

This repo uses [Cocogitto](https://docs.cocogitto.io/) for conventional commit enforcement and version management.

**Git hook:** A `commit-msg` hook enforces conventional commits. Install it after cloning:
```bash
cog install-hook commit-msg
```

**Commit format:**
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**Version bumping:**
```bash
cog bump --auto          # Auto-detect bump type from commits
cog bump --patch         # Bump patch version (0.1.12 -> 0.1.13)
cog bump --minor         # Bump minor version (0.1.12 -> 0.2.0)
cog bump --major         # Bump major version (0.1.12 -> 1.0.0)
cog bump --dry-run       # Preview what would happen
```

**Other useful commands:**
```bash
cog check                # Verify all commits follow conventional format
cog changelog            # Generate changelog
cog get-version          # Show current version
```

---
> Source: [barryw/PaperlessMCP](https://github.com/barryw/PaperlessMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
