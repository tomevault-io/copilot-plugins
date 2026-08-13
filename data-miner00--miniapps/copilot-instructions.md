## miniapps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project intent

MiniApps is a collection of small, standalone desktop/console apps, each in its own top-level project folder and added to `MiniApps.sln`. Per the README:

- Each app should stay under ~1000 lines of code.
- This is **not** a sandbox/experimentation repo — don't add code just to try things out. Keep each project lean and small.
- When adding a new app, create a new project folder + `.csproj` and add it to `MiniApps.sln`, following the existing pattern (see `ODataQuerySimulator` for a console app, `Borderless` for a WPF app).

## Build & run

Requires .NET 10 SDK and (for WPF projects) Windows.

```
dotnet build                                    # build entire solution
dotnet build MiniApps.sln -c Release             # matches CI (azure-pipelines.yml)
dotnet run --project ODataQuerySimulator         # run the OData console app
dotnet run --project CronExpressionSimulator     # run the cron expression console app
dotnet run --project Borderless                  # run the borderless WPF app
```

There are no test projects in this repo currently.

CI (`azure-pipelines.yml`) runs on an Azure Pipelines self-hosted (`default` pool) agent, on push to `master`: NuGet restore + `VSBuild` of `MiniApps.sln`, `Release` config, `Any CPU`.

## Projects

- **Borderless** (`Borderless/`) — WPF app (`net10.0-windows`), `OutputType=WinExe`. A single borderless, non-resizable window (`WindowStyle="None"`, `ResizeMode="NoResize"`) that displays an image (`Image/dawn.jpg`) and is dragged by left-click-and-drag on the window body (`Window_MouseDown` → `DragMove()`). Minimal single-window/single-code-behind structure (`MainWindow.xaml` + `MainWindow.xaml.cs`), no MVVM/DI.
- **ODataQuerySimulator** (`ODataQuerySimulator/`) — Console app (`net10.0`) built on `Microsoft.OData.Core`/`Microsoft.OData.Edm`. Everything lives in `Program.cs`:
  - `EdmModelBuilder` defines an in-memory EDM model for a `Product` entity (mirrors the `Product` POCO's properties).
  - `FilterEvaluator` walks the OData AST (`ODataQueryOptionParser(...).ParseFilter()`) and evaluates `$filter` expressions in-memory against a hardcoded `List<Product>`, without needing an actual OData service. Supports `and`/`or`/`not`, comparison operators, and the string functions `contains`/`startswith`/`endswith`/`tolower`/`toupper`/`trim`/`length`.
  - The `Program.Main` loop is a REPL: reads a `$filter=` expression from stdin, parses+evaluates it, and prints matches; supports `help`, `data`, `exit`/`quit` commands.
  - When extending supported OData syntax, add cases to `EvalBinary`/`EvalFunction`/`ResolveFunctionValue` in `FilterEvaluator` — unsupported node types/operators/functions throw `NotSupportedException` by design (caught and reported in the REPL loop), so no silent fallback behavior is expected.
- **CronExpressionSimulator** (`CronExpressionSimulator/`) — Console app (`net10.0`) built on the [Cronos](https://github.com/HangfireIO/Cronos) NuGet package for cron parsing/scheduling. Everything lives in `Program.cs`: a REPL (`cron>` prompt) that parses a cron expression via `CronExpression.Parse` and prints the next 10 occurrences. Supports standard 5-field expressions, an optional leading seconds field (auto-detected by field count in `ParseFormat`), and `@` macros (`@daily`, `@hourly`, etc.). `GetNextOccurrence` requires the `from` timestamp to be UTC when a `TimeZoneInfo` is supplied — internal timestamps are kept in UTC and only converted to local time for display. Supports `help`/`exit`/`quit` commands.

---
> Source: [data-miner00/MiniApps](https://github.com/data-miner00/MiniApps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
