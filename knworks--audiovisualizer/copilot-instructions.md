## audiovisualizer

> `AudioVisualizer.slnx` is the entry point for all projects. Keep non-UI logic in `AudioVisualizer.Core/`, WPF controls and themes in `AudioVisualizer.Wpf/`, and manual verification code in `AudioVisualizer.SampleApp/`. Tests belong in the matching test projects: `AudioVisualizer.Core.Tests/` and `AudioVisualizer.Wpf.Tests/`. Requirements and design notes live in `docs/`, currently centered on `docs/01_requirements.md`. Do not commit generated output from `bin/`, `obj/`, or local metadata such as `.serena/`.

# Repository Guidelines

## Project Structure & Module Organization

`AudioVisualizer.slnx` is the entry point for all projects. Keep non-UI logic in `AudioVisualizer.Core/`, WPF controls and themes in `AudioVisualizer.Wpf/`, and manual verification code in `AudioVisualizer.SampleApp/`. Tests belong in the matching test projects: `AudioVisualizer.Core.Tests/` and `AudioVisualizer.Wpf.Tests/`. Requirements and design notes live in `docs/`, currently centered on `docs/01_requirements.md`. Do not commit generated output from `bin/`, `obj/`, or local metadata such as `.serena/`.

## Build, Test, and Development Commands

Use the .NET 10 SDK on Windows.

- `dotnet build AudioVisualizer.slnx` builds the full solution.
- `dotnet test AudioVisualizer.slnx --no-build` runs the configured NUnit test projects after a successful build.
- `dotnet run --project AudioVisualizer.SampleApp/AudioVisualizer.SampleApp.csproj` launches the sample WPF app.
- `dotnet format AudioVisualizer.slnx --verify-no-changes` checks formatting without rewriting files.

## Coding Style & Naming Conventions

Follow the existing C# style: 4-space indentation, block-scoped namespaces, and standard .NET naming. Use `PascalCase` for public types and members, keep namespaces under `AudioVisualizer.*`, and preserve nullable-aware code because `Nullable` is enabled across projects. Keep WPF-specific resources in `Themes/Generic.xaml` and avoid leaking WPF types into `AudioVisualizer.Core`.

## Testing Guidelines

The repository uses NUnit with `Microsoft.NET.Test.Sdk`, `NUnit3TestAdapter`, and `coverlet.collector`. Add new tests to the corresponding `*.Tests` project and name files after the unit under test, for example `SpectrumRendererTests.cs`. Prefer method names that describe behavior, such as `Start_WhenDeviceMissing_Throws`. At the moment, the test projects are configured but no discoverable test cases are checked in yet.

## Commit & Pull Request Guidelines

Recent history uses short summaries, often in Japanese. For new work, prefer Conventional Commits such as `feat(wpf): add visualizer control` or `docs: update contributor guide`. Pull requests should explain the scope, list affected projects, note the commands used for verification, and include screenshots or short recordings for visible WPF changes. Link the relevant issue or requirement section when available.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Knworks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
