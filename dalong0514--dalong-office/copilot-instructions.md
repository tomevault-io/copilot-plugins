## dalong-office

> The solution lives under `office-tools/office-tools.sln`. The main console app resides in `office-tools/office-tools/`, with `Program.cs` as the entry point targeting .NET 8.0. Keep generated `bin/` and `obj/` folders out of reviews, and group new source files under `office-tools/office-tools/` following feature-based subfolders (for example `Commands/Export`). Shared assets should land in a new `Assets/` directory alongside the project file.

# Repository Guidelines

## Project Structure & Module Organization
The solution lives under `office-tools/office-tools.sln`. The main console app resides in `office-tools/office-tools/`, with `Program.cs` as the entry point targeting .NET 8.0. Keep generated `bin/` and `obj/` folders out of reviews, and group new source files under `office-tools/office-tools/` following feature-based subfolders (for example `Commands/Export`). Shared assets should land in a new `Assets/` directory alongside the project file.

## Build, Test, and Development Commands
- `dotnet restore` - install project dependencies; run after pulling new packages.
- `dotnet build office-tools/office-tools.sln` - compile and validate the solution.
- `dotnet run --project office-tools/office-tools` - execute the CLI locally.
- `dotnet test` - run the full test suite once a test project is added.
- `dotnet format` - apply formatting and style fixes before submitting.

## Coding Style & Naming Conventions
Use four-space indentation and the default C# brace style. Name classes, records, and public methods with PascalCase; locals and parameters use camelCase. Files should match the primary type name. The root namespace is currently `office_tools`; align new namespaces with it until we formalize a rename. Enable nullable annotations in new code and prefer expression-bodied members when they improve readability.

## Testing Guidelines
Add tests under a future `office-tools/tests/office-tools.Tests` project using xUnit. Mirror the source folder structure to keep context clear. Name test classes `<TypeUnderTest>Tests` and methods `<Scenario>_<Expectation>`. Run `dotnet test` locally before opening a pull request and aim to cover new branches or edge cases introduced.

## Commit & Pull Request Guidelines
Recent commits are brief (`Initial commit`, `新建项目文件`); continue using concise, imperative subjects (<= 50 chars) and include a body when behavior changes. Prefer English or provide an English summary for shared understanding. Pull requests should describe the change, link related issues, and list the commands run (for example `dotnet build`, `dotnet test`). Attach screenshots or console output when altering user-facing behavior.

---
> Source: [dalong0514/dalong.office](https://github.com/dalong0514/dalong.office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
