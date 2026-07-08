## pure-relationalschema-storage-abstractions-serialization-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All `dotnet` commands must be run from the `./src` directory.

```bash
dotnet restore
dotnet build --no-restore -warnaserror
dotnet test --no-build
dotnet format --verify-no-changes             # check code style (CI enforces this)
dotnet format && csharpier format .           # auto-fix code style
dotnet pack --configuration Release -p:PackageVersion=<version> --output .
```

## Architecture

This is a **serialization library** — it provides `System.Text.Json` converters for the `Pure.RelationalSchema.Storage` domain interfaces. There are no abstractions or business logic defined here; the only purpose is to make `ICell`, `IRow`, `IStoredTableDataSet`, and `IStoredSchemaDataSet` round-trip cleanly through JSON.

**Public API surface** (namespace `Pure.RelationalSchema.Storage.Abstractions.Serialization.System`):

- `RelationalSchemaStorageConverters` — sealed class implementing `IEnumerable<JsonConverter>`; yields all four converters as the single entry point for registration
- `CellConverter : JsonConverter<ICell>`
- `RowConverter : JsonConverter<IRow>`
- `StoredTableDataSetConverter : JsonConverter<IStoredTableDataSet>`
- `StoredSchemaDataSetConverter : JsonConverter<IStoredSchemaDataSet>`

Each converter uses an internal DTO record for JSON mapping; deserialized results are concrete records from `Pure.RelationalSchema.Storage` (`Cell`, `Row`, `StoredSchemaDataset`) or an internal `StoredTableDataSetFromJsonModel` that implements `IStoredTableDataSet` and `IQueryable<IRow>`.

**Multi-targeting:** net8.0, net9.0, net10.0. `IsAotCompatible` is `false`.

**Package validation:** `EnablePackageValidation = true` with `PackageValidationBaselineVersion = 0.1.0-preview.0.2.1`. Breaking API changes fail the build.

**Publishing:** triggered by pushing a semver tag (pattern `*.*.*`). The tag becomes the `PackageVersion`.

**Tests:** xUnit project at `src/Tests/Pure.RelationalSchema.Storage.Abstractions.Serialization.System.Tests/`, targeting net10.0. CI runs mutation testing via dotnet-stryker with a 95% threshold and enforces ≥95% line coverage (warning at 99%).

## Code Style

Enforced via `.editorconfig` and `dotnet format --verify-no-changes` in CI:

- No `var` — always use explicit types
- No expression-bodied methods or constructors; expression-bodied properties and accessors are required
- File-scoped namespace declarations (`namespace Foo.Bar;`)
- `csharp_new_line_before_open_brace = all` — opening braces always on a new line
- Private fields: `_camelCase` prefix
- Max line length: 90 characters
- `using` directives outside the namespace

## Commit Messages

Do not mention Claude or AI assistance in commit messages.

---
> Source: [kudima03/Pure.RelationalSchema.Storage.Abstractions.Serialization.System](https://github.com/kudima03/Pure.RelationalSchema.Storage.Abstractions.Serialization.System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
