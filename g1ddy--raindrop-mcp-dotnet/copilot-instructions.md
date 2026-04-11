## raindrop-mcp-dotnet

> This document provides instructions for AI agents working with this repository.

# Agent Instructions

This document provides instructions for AI agents working with this repository.

## Code Layout

This project is a .NET-based MCP server for Raindrop.io. The code is structured by resource type.

-   **`src/Mcp/Program.cs`**: The application entry point. It configures the DI container and starts the MCP host.
-   **`src/Mcp/RaindropServiceCollectionExtensions.cs`**: Registers all the services for the application.
-   **`src/Mcp/{Resource}`**: Each resource (e.g., `Collections`, `Tags`, `Raindrops`) has its own directory containing:
    -   **`I{Resource}Api.cs`**: The Refit interface for the Raindrop.io API.
    -   **`{Resource}Tools.cs`**: The MCP tools for that resource. Methods in these files are decorated with the `[McpServerTool]` attribute.
    -   **Models**: C# classes that represent the data structures for that resource.
-   **`src/Mcp/Common`**: Contains shared models and base classes.
-   **`src/Mcp/Prompts`**: Contains pre-defined prompts for common workflows.

## Development Environment and Validation

The development environment is pre-configured with the .NET 8 SDK. To restore dependencies, build the project, and run all tests, you can use the following commands:

```bash
dotnet restore RaindropMcp.sln
dotnet build RaindropMcp.sln --no-restore
dotnet test RaindropMcp.sln --no-build --no-restore
```

### Environment Setup

This project uses a pre-baked Jules VM snapshot. If you need to reproduce the environment setup or understand how it was configured, refer to `scripts/setup.sh`. This script handles the installation of the .NET SDK and package restoration. See `scripts/README.md` for more details.

## Testing and Modifying Request/Response Objects

When modifying C# models or integration tests, strict adherence to the Raindrop.io API specification is required. The API behavior often differs from simple CRUD conventions.

### Request Models and Field Mapping

- **API Documentation as Source of Truth**: Always verify field names and types against [Raindrop.io API V1 Documentation](https://developer.raindrop.io/v1). Do not guess based on existing C# properties.
- **Complex Objects**: Some flat C# properties must be mapped to nested objects in the API JSON.
    - *Example*: `CollectionId` (int) in `IRaindropRequest` must be mapped to `collection: { "$id": 123 }` in the API DTO. Mapping it to a top-level `collectionId` field will fail for Create/Update operations.
    - Use `Mcp.Common.IdRef` to serialize properties with the `$id` key (e.g., `[JsonPropertyName("$id")]`).
- **Forbidden Fields**: Request DTOs should omit read-only fields (e.g., `user`, `broken`) unless explicitly allowed by the `POST`/`PUT` endpoint. Use `[JsonIgnore]` or separate DTO classes (e.g., `RaindropCreateRequest` vs `Raindrop` response model) to enforce this.

### Integration Testing (VCR)

- **VCR Workflow**:
    - **Replay Mode (Default)**: Uses existing JSON fixtures in `tests/Mcp.Tests/Integration/Fixtures`. No API token is required.
    - **Record Mode**: Activated when a fixture is missing AND a valid `Raindrop:ApiToken` is present.
- **Robust Cleanup**:
    - Wrap *each* cleanup step in a `try/catch` block within the `finally` clause.
    - *Reasoning*: If the first cleanup action (e.g., `DeleteTag`) fails, subsequent actions (e.g., `DeleteBookmark`) must still execute to prevent resource leaks in the live API environment.
- **CI Workflow**:
    - The CI workflow uses `git status --porcelain` to detect changes.
    - *Critical*: `git diff` does not detect untracked files. Since VCR recording creates *new* JSON files, the workflow must use `git status` to verify that no new recordings were unexpectedly generated during a test run (which would indicate non-determinism or leakage).

## Performance and Complexity Guidelines

- **Prioritize Readability**: Code should be straightforward and maintainable. Avoid complex optimizations (e.g., manual array manipulations, `StringBuilder` for small concatenations, `stackalloc`) unless there is a proven, significant performance bottleneck.
- **Micro-optimizations**: Avoid micro-optimizations that provide negligible or inconclusive gains but increase code complexity or add many lines of code.
- **LINQ vs. Imperative**: Use simple LINQ statements over manual loops where it improves clarity, unless working in a high-frequency, performance-critical path (e.g., inside a large loop).
- **Simple Idioms**: Favor standard C# idioms (like string interpolation and `string.Join`) for UI/message generation tasks where the performance impact is dwarfed by I/O.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g1ddy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/g1ddy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
