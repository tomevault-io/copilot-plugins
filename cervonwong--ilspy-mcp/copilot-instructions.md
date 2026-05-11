## ilspy-mcp

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**ILSpy MCP — Feature Parity**

An MCP (Model Context Protocol) server that exposes ILSpy's .NET decompilation and static analysis capabilities as tools for AI assistants. Currently has 8 tools covering basic type inspection (~25-30% of ILSpy GUI functionality). This milestone extends it to full reverse engineering feature parity — cross-reference tracing, IL output, assembly metadata, string search, resource extraction, and bulk decompilation.

**Core Value:** AI assistants can perform complete .NET static analysis workflows — not just read code, but trace execution, find usages, search strings, and navigate across types and assemblies.

### Constraints

- **Tech stack**: C#, .NET, ICSharpCode.Decompiler, System.Reflection.Metadata, MCP SDK — no new runtime dependencies
- **Architecture**: Follow existing layered pattern (Domain/Infrastructure/Application/Transport)
- **Testing**: Critical-path tests for P0 features and all bug fixes to ensure nothing breaks
- **Compatibility**: Must not break existing 8 tools during upgrades
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Current Stack (Baseline)
| Technology | Current Version | Target Version | Breaking? |
|------------|----------------|----------------|-----------|
| ICSharpCode.Decompiler | 9.1.0.7988 | 10.0.0.8330 | YES |
| ModelContextProtocol | 0.4.0-preview.3 | 1.2.0 | YES |
| Microsoft.Extensions.Hosting | 8.0.0 | 10.0.0 | Minor |
| .NET Runtime | net9.0 | net9.0 (keep) | NO |
| xUnit | 2.9.2 | 2.9.3 (keep v2) | NO |
| FluentAssertions | 8.8.0 | 8.9.0 | NO |
## Recommended Stack
### Core Decompilation
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| ICSharpCode.Decompiler | 10.0.0.8330 | C# decompilation, IL disassembly, type system | Just released (2026-04-06). Still targets netstandard2.0, compatible with net9.0. Adds `IDecompilerTypeSystem` interface on `CSharpDecompiler` constructor for better testability, new `ExpandParamsArguments` and `AlwaysMoveInitializer` settings. |
| System.Reflection.Metadata | 9.0.0+ (transitive) | IL scanning, metadata reading, cross-reference analysis | Comes transitively via ICSharpCode.Decompiler. Provides `MetadataReader`, `MethodBodyBlock.GetILReader()`, `ILOpCode` enum, `BlobReader` for raw IL bytecode scanning. This is the engine for string search (`ldstr`), constant search (`ldc.*`), and cross-reference tracing. |
### MCP Server
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| ModelContextProtocol | 1.2.0 | MCP server framework with tool registration | Stable release (GA since Feb 2025). The `[McpServerToolType]` / `[McpServerTool]` attribute pattern and `WithToolsFromAssembly()` builder survive from 0.4.0-preview -- the core pattern used in this project is preserved. |
| ModelContextProtocol.Core | 1.2.0 (transitive) | Core protocol types | Pulled in automatically by ModelContextProtocol package. |
| Microsoft.Extensions.Hosting | 10.0.0 | DI, configuration, lifecycle management | Required by MCP SDK 1.2.0 (depends on Microsoft.Extensions.Hosting.Abstractions >= 10.0.5). Must upgrade from 8.0.0. |
| Microsoft.Extensions.Logging.Console | 10.0.0 | Stderr logging for MCP transport | Keep aligned with Hosting version. |
### Testing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| xUnit | 2.9.3 | Test framework | Stay on v2. xUnit v3 (3.2.2) is stable but requires project restructuring (test projects become executables, package renames to `xunit.v3`, attribute API changes). Not worth the migration churn during a feature milestone. Upgrade separately later. |
| FluentAssertions | 8.9.0 | Assertion library | Minor patch update. No breaking changes from 8.8.0. |
| Microsoft.NET.Test.Sdk | 17.12.0 | Test runner infrastructure | Keep current. Compatible with xUnit 2.x on net9.0. |
| coverlet.collector | 6.0.2 | Code coverage | Keep current. Works with xUnit 2.x. |
### Infrastructure (No New Dependencies)
| Technology | Version | Purpose | Why NOT add new packages |
|------------|---------|---------|--------------------------|
| System.Reflection.Metadata | (transitive) | IL scanning for cross-refs, string search, constant search | Already a dependency of ICSharpCode.Decompiler. Provides everything needed: `MetadataReader`, `MethodBodyBlock`, `BlobReader`, `ILOpCode`. No wrapper library needed. |
| ICSharpCode.Decompiler.Disassembler | (built-in namespace) | IL/CIL output for types and methods | Part of ICSharpCode.Decompiler package. `ReflectionDisassembler` class with `DisassembleType()` and `DisassembleMethod()`. No additional package. |
## Breaking Changes: Migration Guide
### ICSharpCode.Decompiler 9.1 to 10.0
| Change | Impact on This Project | Action Required |
|--------|----------------------|-----------------|
| Removed `ITypeReference` and implementations | LOW -- Project uses `FullTypeName` and `ITypeDefinition`, not `ITypeReference` | Verify no transitive usage in type hierarchy code |
| `ResolvedUsingScope` renamed to `UsingScope` | NONE -- Project does not use using scope APIs | No action |
| Removed `UnresolvedUsingScope` | NONE -- Not used | No action |
| Removed `ToTypeReference` | LOW -- Check if `FindTypeHierarchyUseCase` or type resolution code uses this | Search codebase for `ToTypeReference` calls |
| `CSharpDecompiler` constructor accepts `IDecompilerTypeSystem` | POSITIVE -- Enables better testability with mock type systems | Opportunity for test improvement, not a break |
| `ILInstruction.Extract()` returns `ILVariable?` (nullable) | LOW -- Only relevant if building IL analysis on top of ILInstruction tree | Relevant for new cross-ref features; handle nulls |
| `MetadataFile` constructor accepts `MetadataStringDecoder` | NONE -- Optional parameter | No action |
| New settings: `ExpandParamsArguments`, `AlwaysMoveInitializer` | NONE -- Additive | Consider enabling for better decompilation output |
### ModelContextProtocol 0.4.0-preview.3 to 1.2.0
| Change | Impact on This Project | Action Required |
|--------|----------------------|-----------------|
| Core builder pattern unchanged | POSITIVE -- `AddMcpServer().WithStdioServerTransport().WithToolsFromAssembly()` still works | Verify compilation |
| `[McpServerToolType]` and `[McpServerTool]` attributes preserved | POSITIVE -- All 8 existing tools use this pattern | No structural change to tool classes |
| `RequestOptions` bag replaces individual parameters | LOW -- Project tools return `string`, not using `JsonSerializerOptions` directly | Check if any tool passes serialization options |
| `IOptions<McpServerHandlers>` removed (v0.9 change) | UNKNOWN -- Check if `Program.cs` configures handlers via DI | Verify server configuration code |
| Binary data types changed to `ReadOnlyMemory<byte>` (v0.9) | NONE -- Project returns text, not binary | No action |
| `Tool.Name` now required property | LOW -- All tools already set `Name` via attribute | No action |
| Legacy SSE endpoints disabled by default (v1.2) | NONE -- Project uses stdio transport | No action |
| Collection types changed `List<T>` to `IList<T>` | LOW -- May affect if code accesses protocol types directly | Check compilation |
### Microsoft.Extensions.Hosting 8.0.0 to 10.0.0
## Alternatives Considered
| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Decompiler | ICSharpCode.Decompiler 10.0 | dnSpy/dnlib | dnSpy is abandoned (archived 2020). dnlib is lower-level -- no C# decompilation, only IL/metadata reading. ICSharpCode.Decompiler provides both. |
| Decompiler | ICSharpCode.Decompiler 10.0 | ICSharpCode.Decompiler 9.1 (stay) | 10.0 is netstandard2.0 compatible, just released, and the project requirements call for the upgrade. Breaking changes are minimal for this codebase. |
| MCP SDK | ModelContextProtocol 1.2.0 | ModelContextProtocol 1.0.0 | 1.2.0 is latest stable with bug fixes. No reason to target an older stable when the upgrade path is the same from 0.4.0-preview. |
| IL scanning | System.Reflection.Metadata (transitive) | Mono.Cecil | Adding Cecil would duplicate functionality already available via S.R.Metadata (which ICSharpCode.Decompiler already depends on). Cecil would add an unnecessary dependency with overlapping capabilities. |
| Testing | xUnit 2.9.x | xUnit 3.2.x | v3 migration requires structural changes (OutputType=Exe, package renames, attribute changes). Not worth it during a feature milestone. |
| Testing | xUnit 2.9.x | NUnit 4.x | Existing tests use xUnit. No reason to switch frameworks mid-project. |
| Assertions | FluentAssertions 8.9.0 | Shouldly | FluentAssertions already in use, well-maintained, wider adoption. |
## Key APIs for New Features
### IL Scanning (cross-refs, string search, constant search)
### IL/CIL Disassembly Output
### Assembly Metadata
### Embedded Resources
## Installation
# Core project -- update existing references
# Test project -- update existing references
## Do NOT Add
| Package | Why Not |
|---------|---------|
| Mono.Cecil | Overlaps with System.Reflection.Metadata already in dependency tree |
| dnlib | Lower-level than needed; ICSharpCode.Decompiler wraps S.R.Metadata already |
| ILSpyX | Contains non-UI analyzers but tightly coupled to ILSpy app model; not designed for library consumption |
| System.Reflection.Metadata (explicit) | Already a transitive dependency via ICSharpCode.Decompiler. Adding explicitly risks version conflicts. |
| Moq / NSubstitute | Not needed for this project's test approach (integration tests against real assemblies). The new `IDecompilerTypeSystem` interface in Decompiler 10.0 enables constructor-based testing without mocking frameworks. |
## Target Framework Decision
## Sources
- [NuGet: ICSharpCode.Decompiler 10.0.0.8330](https://www.nuget.org/packages/ICSharpCode.Decompiler/10.0.0.8330) -- package metadata, dependencies, target framework
- [NuGet: ModelContextProtocol 1.2.0](https://www.nuget.org/packages/ModelContextProtocol/1.2.0) -- package metadata, dependencies
- [GitHub: ILSpy Releases](https://github.com/icsharpcode/ILSpy/releases) -- breaking changes across 9.1 to 10.0
- [GitHub: MCP C# SDK Releases](https://github.com/modelcontextprotocol/csharp-sdk/releases) -- breaking changes across 0.4.0 to 1.2.0
- [.NET Blog: MCP C# SDK v1.0](https://devblogs.microsoft.com/dotnet/release-v10-of-the-official-mcp-csharp-sdk/) -- v1.0 announcement and API patterns
- [MCP C# SDK Docs](https://csharp.sdk.modelcontextprotocol.io/concepts/getting-started.html) -- getting started, builder pattern
- [Microsoft Learn: MethodBodyBlock.GetILReader](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata.methodbodyblock.getilreader) -- IL scanning API
- [Microsoft Learn: ILOpCode](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.metadata.ilopcode) -- IL opcode enumeration
- [xUnit v3 Migration Guide](https://xunit.net/docs/getting-started/v3/migration) -- v2 to v3 changes (deferred)
- [NuGet: FluentAssertions 8.9.0](https://www.nuget.org/packages/fluentassertions/) -- latest version
- [NuGet: xUnit v3 3.2.2](https://www.nuget.org/packages/xunit.v3) -- available but deferred
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [cervonwong/ILSpy-MCP](https://github.com/cervonwong/ILSpy-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
