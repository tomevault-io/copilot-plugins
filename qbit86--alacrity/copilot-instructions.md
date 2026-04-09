## alacrity

> - Build solution: `dotnet build Alacrity.sln`

# Alacrity Development Guidelines

## Build Commands

- Build solution: `dotnet build Alacrity.sln`
- Run application: `dotnet run --project src/Alacrity.App/Alacrity.App.csproj`
- Build release package: `dotnet pack Alacrity.sln -c Release`
- Run tests: `dotnet test Alacrity.sln`
- Run single test: `dotnet test --filter "FullyQualifiedName~TestName" Alacrity.sln`

## Code Style

- Use file-scoped namespaces
- 4-space indentation, 120 character line limit
- UTF-8 encoding with final newline
- PascalCase for public members, camelCase for parameters/locals
- Private fields: `_camelCase`, static: `s_camelCase`
- Enable nullable reference types
- Use `var` when type is apparent
- Prefer expression-bodied members when single line
- Use TryPattern (`bool Try(out Result)`) for error handling
- Return `TryHelpers.Some/None` for consistent Try patterns
- Organize imports alphabetically
- Prefer static local functions
- Make fields readonly whenever possible
- Use [MethodImpl(MethodImplOptions.AggressiveInlining)] judiciously

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qbit86)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qbit86)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
