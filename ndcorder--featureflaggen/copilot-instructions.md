## featureflaggen

> - **Source Generator** targeting `netstandard2.0` (required by Roslyn)

# FeatureFlagGen — Project Conventions

## Architecture
- **Source Generator** targeting `netstandard2.0` (required by Roslyn)
- **Tests** targeting `net9.0` using xUnit
- Incremental source generator using `IIncrementalGenerator`

## Coding Style
- C# with nullable reference types enabled
- File-scoped namespaces
- `LangVersion: latest`
- XML doc comments on public APIs only
- No `#region` blocks
- Prefer pattern matching and expression-bodied members where clear

## Namespace
- `FeatureFlagGen` for the generator
- `FeatureFlagGen.Tests` for tests

## Commit Format
- `feat:` new features
- `fix:` bug fixes
- `test:` test additions/changes
- `docs:` documentation
- `chore:` project config, CI
- `refactor:` code cleanup

## Test Patterns
- Use Roslyn `CSharpGeneratorDriver` for source generator testing
- Test helper methods provide compilation setup
- Assert on generated source text and diagnostics
- Name tests: `MethodUnderTest_Scenario_ExpectedResult`

## Build & Test
```bash
dotnet build
dotnet test
dotnet format --verify-no-changes  # lint check
```

---
> Source: [ndcorder/FeatureFlagGen](https://github.com/ndcorder/FeatureFlagGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
