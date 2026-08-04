## systemvisualizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SystemVisualizer is a .NET 9 cross-platform desktop application built with Avalonia UI for visualizing system graphs. It reads electrical panel data from Excel/CSV files and displays them as interactive node graphs with automatic layout algorithms.

## Build Commands

```bash
# Build solution
dotnet build src/SystemVisualizer.sln

# Run desktop application
dotnet run --project src/SystemVisualizer.Desktop/SystemVisualizer.Desktop.csproj

# Run all tests
dotnet test src/SystemVisualizer.sln

# Run specific test project
dotnet test src/SystemVisualizer.Tests/SystemVisualizer.Tests.csproj

# Build Browser (WASM) project for local testing
dotnet publish src/SystemVisualizer.Browser/SystemVisualizer.Browser.csproj -c Release -o publish
```

## Browser Deployment (GitHub Pages)

The app is deployed to GitHub Pages at: https://pae-engineers.github.io/SystemVisualizer/

### Deployment Workflow

The `.github/workflows/deploy-github-pages.yml` workflow automatically deploys to GitHub Pages on push to `main`.

**Required workloads:** The workflow installs `wasm-tools` and `wasm-tools-net9` for .NET WASM compilation.

### IL Trimming Configuration

The Browser project uses IL trimming for smaller WASM bundles. To prevent reflection-based libraries from breaking, assemblies are preserved in `SystemVisualizer.Browser.csproj`:

```xml
<ItemGroup>
  <TrimmerRootAssembly Include="CsvHelper" />
  <TrimmerRootAssembly Include="SystemVisualizer.DataProviders" />
</ItemGroup>
```

### Local Browser Testing

```bash
# Option 1: Direct dotnet serve
dotnet publish src/SystemVisualizer.Browser -c Release -o publish
# Then serve publish/wwwroot with any static file server

# Option 2: Docker (requires Docker Desktop)
docker-compose -f docker-compose.serve.yml up -d
# Access at http://localhost:3000
```

## Architecture

### Project Structure

- **SystemVisualizer** - Main Avalonia UI application (Views, ViewModels, Controls, Themes)
- **SystemVisualizer.Core** - Domain models and interfaces (Node, Edge, Connector, Graph, IDataProvider, ILayoutProvider)
- **SystemVisualizer.DataProviders** - Excel/CSV file parsing using ExcelMapper and CsvHelper
- **SystemVisualizer.Layout** - Graph layout computation using Microsoft MSAGL
- **SystemVisualizer.Desktop** - Desktop entry point executable
- **SystemVisualizer.Browser** - Browser (WASM) entry point for web deployment
- **SystemVisualizer.Tests** - NUnit test project

### Key Patterns

**MVVM Architecture:**
- ViewModels use CommunityToolkit.MVVM with `[ObservableProperty]` and `[RelayCommand]` attributes
- Views are XAML-based (`.axaml` files) with compiled bindings (`x:DataType`)
- WeakReferenceMessenger for decoupled communication between components

**Data Flow:**
1. User opens file → `MainViewModel.OpenDataFileAsync()`
2. `ExcelDataProvider.GetNodes()` parses data into nodes and connections
3. `EditorViewModel.LoadData()` populates observable collections
4. `MsaglLayoutProvider.ApplyLayout()` positions nodes
5. Views bind to collections and render the graph

### Key Interfaces

- **IGraphItem** - Node representation with Name, Location, Bounds, Cluster
- **IDataProvider** - Loads graph data from files (streams)
- **ILayoutProvider** - Computes node positions for given nodes/connections
- **ILayoutCalculator** - Layout algorithm abstraction (Sugiyama, MDS, Ranking, etc.)

## Technology Stack

- .NET 9.0 with nullable reference types
- Avalonia UI 11.3.0 (cross-platform)
- Nodify/NodifyAvalonia for node editor control
- Microsoft.Msagl for hierarchical graph layout
- ExcelMapper/CsvHelper for data import
- CommunityToolkit.Mvvm for MVVM infrastructure
- NUnit 4.3 for testing

## Configuration

- Central package management via `Directory.Packages.props`
- .NET SDK 9.0 with pre-release allowance (`global.json`)
- Debug builds include Avalonia.Diagnostics

---
> Source: [PAE-Engineers/SystemVisualizer](https://github.com/PAE-Engineers/SystemVisualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
