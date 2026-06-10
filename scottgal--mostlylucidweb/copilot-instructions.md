## mostlylucidweb

> - **Full build**: `dotnet restore && dotnet build Mostlylucid.sln` (net9.0)

# Repository Guidelines for Agentic Coding

## Build & Test Commands
- **Full build**: `dotnet restore && dotnet build Mostlylucid.sln` (net9.0)
- **Run site**: `dotnet run --project Mostlylucid/Mostlylucid.csproj`
- **Front-end**: `cd Mostlylucid && npm install && npm run dev`
- **Single test**: `dotnet test Mostlylucid.Test/ --filter "TestName"`
- **Coverage**: `dotnet test Mostlylucid.sln --collect:"XPlat Code Coverage"`

## Coding Style
- **C#**: File-scoped namespaces, 4-space indent, PascalCase types, camelCase locals
- **Async**: Suffix with `Async`, use expression-bodied members
- **Imports**: Prefer `var` for obvious types, keep using statements concise
- **Error handling**: Use try-catch with specific exceptions, log failures appropriately
- **JavaScript**: ESM imports, small modules, mirror existing naming patterns

## Testing
- **xUnit**: Name files `*Tests.cs`, keep fixtures near code
- **Front-end**: Vitest + Happy DOM, use data attributes for assertions
- **Always test locally** before claiming completion

## Commit Guidelines
- Conventional commits: `feat:`, `fix:`, `chore:`
- Include test evidence and rationale for config/infra changes
- Never commit secrets (use local overrides)

---
> Source: [scottgal/mostlylucidweb](https://github.com/scottgal/mostlylucidweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
