## resty

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Resty is a local-first, text-driven HTTP API client for Windows. `.http` files (JetBrains HTTP syntax) are the single source of truth — plain text, Git-trackable, shared between GUI and CLI. Three components:

- **Resty.Core** — pure .NET 10 library, zero external dependencies. Parsing, execution, assertions, environment variables, reporting.
- **Resty.Cli** — thin CLI wrapper (`resty run` / `resty test`), NativeAOT-capable.
- **Resty.Gui** — Windows desktop GUI using MewUI (native Direct2D rendering, not Electron).

## Build & Run

```bash
# Prerequisite: .NET 10 SDK

# Launch GUI
dotnet run --project src/Resty.Gui/

# CLI: execute requests
dotnet run --project src/Resty.Cli/ -- run samples/smoke.http --env dev

# CLI: run assertions (exit code 1 on failure)
dotnet run --project src/Resty.Cli/ -- test samples/smoke.http --env dev --report junit

# Run tests
dotnet test tests/Resty.Core.Tests/

# NativeAOT single-file publish (requires VS Build Tools)
.\publish-aot.cmd
```

## Architecture

### Layered design (strict, no circular dependencies)

```
Resty.Gui ──→ Resty.Core
Resty.Cli ──→ Resty.Core
```

`Resty.Core` has zero NuGet dependencies — uses only BCL (`System.Net.Http`, `System.Text.Json`). `Resty.Cli` depends only on `Resty.Core`. `Resty.Gui` depends on `Resty.Core` + `Aprillz.MewUI.Windows`.

### Data flow

```
.http file → HttpFileParser → HttpFileDefinition
  → EnvironmentResolver ({{var}} substitution)
  → HttpRequestExecutor → HttpExecutionResult
  → AssertionEngine → List<AssertionResult>
  → IReporter → output (text/json/junit)
```

Both CLI and GUI use this same pipeline. The GUI calls Core directly, not via CLI process.

### GUI architecture

VS Code-inspired layout built on MewUI's `ObservableValue<T>` data binding (no Rx.NET, no MVVM framework):

- `MainWindow` owns tab lifecycle and a `_tabStateCache` dictionary to preserve editor state when tabs are switched.
- `WorkspaceService` scans `*.http` files, maintains in-memory parsed cache, watches filesystem with `FileSystemWatcher` (600ms debounce).
- `RequestEditorView` supports dual-mode: Raw text tab ↔ structured tabs (Params/Headers/Auth/Body/Assertions), with bidirectional sync.
- `NativeCustomWindow` extends MewUI's `Window` with DWM borderless custom chrome.
- Views are in `src/Resty.Gui/Views/`, services in `src/Resty.Gui/Services/`.

### Core namespaces

| Namespace | Purpose |
|-----------|---------|
| `Parsing` | `HttpFileParser` (state-machine .http parser), `AssertionParser` (DSL), `CurlConverter` |
| `Execution` | `HttpRequestExecutor` — wraps `HttpClient`, handles timeout/cancellation |
| `Assertions` | `AssertionEngine` — evaluates `status`, `responseTime`, `body.$jsonpath`, `header.Name` operators |
| `Environment` | `EnvironmentResolver` — loads `http-client.env.json` + `http-client.private.env.json`, resolves `{{var}}` |
| `Reporting` | `IReporter` → `TextReporter` (ANSI-colored console), `JsonReporter`, `JUnitReporter` |
| `Models` | Immutable records: `HttpFileDefinition`, `HttpRequestDefinition`, `HttpExecutionResult`, `AssertionRule`, `AssertionResult` |

### Environment variable priority

`private.env.json` > `env.json` > file-level `@variable`

### CLI exit codes

- `0` — all assertions passed
- `1` — assertion failures
- `2` — network errors
- `3` — argument/parse errors

## Testing

xUnit 2.9, tests live in `tests/Resty.Core.Tests/`. Core is the only tested project (no GUI tests). Run a single test:

```bash
dotnet test tests/Resty.Core.Tests/ --filter "FullyQualifiedName~TestClassName"
```

## Key conventions

- All projects use `<Nullable>enable</Nullable>`, `<ImplicitUsings>enable</ImplicitUsings>`.
- Models are immutable (`record` / `record struct` with `init`-only properties).
- `InvariantGlobalization=true` globally — no ICU dependencies (AOT-friendly).
- The GUI has no `.resx` or XAML — all UI is constructed programmatically in C# via MewUI.
- No GitHub Actions / CI configured in repo (no `.github/` directory).
- Design docs live in `docs/` (PRD.md, PRD-GUI.md, UI-Design.md).

---
> Source: [xkyii/Resty](https://github.com/xkyii/Resty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
