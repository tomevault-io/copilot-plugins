## evidence-timeline

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a .NET 10.0 WPF (Windows Presentation Foundation) application for evidence timeline management, targeting Windows platforms. The project follows the MVVM (Model-View-ViewModel) pattern for separating UI concerns from business logic.

## Development Commands

### Building and Running
- `dotnet restore` — Restore NuGet dependencies
- `dotnet build evidence_timeline.csproj` — Compile the application (add `-c Release` for release builds)
- `dotnet run --project evidence_timeline.csproj` — Launch the WPF application
- `dotnet clean` — Remove build artifacts from `bin/` and `obj/`
- `dotnet format` — Apply consistent C# formatting

### Testing
- No test suite currently exists
- When adding tests, use xUnit in a `tests/` folder (e.g., `EvidenceTimeline.Tests`)
- Run tests with `dotnet test`
- Test naming convention: `MethodName_Scenario_ExpectedResult`

## Architecture and Code Organization

### Project Structure
The codebase follows a standard WPF MVVM structure:

- **Entry Points**: `App.xaml`/`App.xaml.cs` (application startup and shared resources), `MainWindow.xaml`/`MainWindow.xaml.cs` (initial window)
- **Recommended Folders** (create as needed):
  - `Views/` — XAML windows and user controls
  - `ViewModels/` — View models implementing INotifyPropertyChanged, commands, and UI state
  - `Models/` — Domain entities and data structures
  - `Services/` — Business logic, data access, and external integrations
  - `Assets/` — Images, icons, and shared XAML styles/resource dictionaries

### MVVM Pattern
- Place UI behavior in view models using commands and data binding
- Keep code-behind minimal (limited to wiring and initialization)
- Shared XAML styles should live in resource dictionaries and be referenced via `App.xaml`

### Target Framework
- .NET 10.0-windows with WPF enabled
- Nullable reference types are enabled project-wide
- Implicit usings are enabled

## Coding Conventions

### Style
- **Indentation**: 4 spaces
- **Braces**: Allman style (newline before `{`)
- **Naming**:
  - PascalCase for types, methods, properties, events
  - camelCase with leading `_` for private fields
  - ALL_CAPS for constants

### Null Safety
- Maintain accurate nullable annotations (`Nullable` is enabled)
- Avoid null-forgiving operator (`!`) unless justified
- Prefer explicit null checks over suppression

### XAML Guidelines
- Use semantic names for elements (e.g., `TimelineList`, `FilterPanel`)
- Keep layouts simple with `Grid` and `StackPanel`
- Extract reusable styles and templates into resource dictionaries

## Commit Guidelines

- Use imperative, concise commit messages (e.g., "Add timeline filtering", "Fix null handling in event import")
- Keep commits focused on a single change
- Separate refactoring from feature changes
- PRs should include:
  - Summary of changes
  - Validation steps (`dotnet build`, `dotnet run`)
  - Screenshots or notes for UI changes
  - Test results when tests exist

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LCRH1883) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
