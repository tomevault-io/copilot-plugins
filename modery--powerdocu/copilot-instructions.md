## powerdocu

> PowerDocu is a .NET 10 Windows application that auto-generates technical documentation for Microsoft Power Platform components (Cloud Flows, Canvas Apps, Model-Driven Apps, Copilot Studio Agents, AI Models, Business Process Flows, Desktop Flows, and Solutions). Output formats are **Word (.docx)**, **Markdown (.md)**, and **HTML**.

# CLAUDE.md — PowerDocu Codebase Guide

## Project Overview

PowerDocu is a .NET 10 Windows application that auto-generates technical documentation for Microsoft Power Platform components (Cloud Flows, Canvas Apps, Model-Driven Apps, Copilot Studio Agents, AI Models, Business Process Flows, Desktop Flows, and Solutions). Output formats are **Word (.docx)**, **Markdown (.md)**, and **HTML**.

The application ships as a **Windows GUI** (`PowerDocu.exe`) and also supports a **CLI** interface via command-line flags.

---

## Repository Structure

```
PowerDocu/
├── PowerDocu.sln                      # Visual Studio solution (all projects)
├── modules/
│   └── PowerDocu.Common/              # Git submodule (https://github.com/modery/PowerDocu.Common)
│       └── PowerDocu.Common/          # The actual library project
├── PowerDocu.GUI/                     # Entry point: WinForms GUI + CLI runner
├── PowerDocu.SolutionDocumenter/      # Orchestrator for .zip solution packages
├── PowerDocu.FlowDocumenter/          # Cloud Flow documentation
├── PowerDocu.AppDocumenter/           # Canvas App documentation
├── PowerDocu.AgentDocumenter/         # Copilot Studio Agent documentation
├── PowerDocu.AIModelDocumenter/       # AI Model documentation
├── PowerDocu.AppModuleDocumenter/     # Model-Driven App documentation
├── PowerDocu.BPFDocumenter/           # Business Process Flow documentation
├── PowerDocu.DesktopFlowDocumenter/   # Desktop Flow (Power Automate Desktop) documentation
├── Images/                            # Screenshots used in README/docs
├── examples/                          # Sample generated output
├── .vscode/                           # VS Code build/launch tasks
├── .github/ISSUE_TEMPLATE/            # GitHub issue templates
├── README.md
├── compile.md                         # How to build from source
├── installation.md                    # Usage instructions
├── settings.md                        # All configuration options
├── roadmap.md                         # Planned features
└── softwarereferences.md              # Third-party library credits
```

---

## Submodule: PowerDocu.Common

**Location:** `modules/PowerDocu.Common/PowerDocu.Common/`  
**Source:** https://github.com/modery/PowerDocu.Common  
**Target:** `net10.0`

After cloning, initialize the submodule:
```bash
git submodule init
git submodule update
```

### Key contents of PowerDocu.Common

**Entity classes** (data models):
- `FlowEntity`, `AppEntity`, `AgentEntity`, `AppModuleEntity`, `AIModel`, `BPFEntity`, `DesktopFlowEntity`
- `SolutionEntity`, `CustomizationsEntity`, `TableEntity`, `ControlEntity`
- `EnvironmentVariableEntity`, `WebResourceEntity`, `OptionSetEntity`, `FormulaDefinitionEntity`

**Parser classes** (extract entities from Power Platform packages):
- `FlowParser`, `AppParser`, `AgentParser`, `SolutionParser`, `CustomizationsParser`
- `BPFXamlParser`, `RobinScriptParser`, `AppActionParser`, `SettingDefinitionParser`, `EnvironmentVariableParser`

**Base builder classes** (shared output generation):
- `WordDocBuilder` — base for all Word document builders
- `MarkdownBuilder` — base for all Markdown builders
- `HtmlBuilder` — base for all HTML builders

**Helper/utility classes**:
- `ConfigHelper` — configuration model (maps to `powerdocu.config.json`)
- `DocumentationContext` — shared state passed across the two-phase pipeline
- `NotificationHelper` — event-driven notification/logging system
- `ConnectorHelper` — connector icon management
- `CharsetHelper` — safe filename/path generation
- `OutputFormatHelper` — output format constants (`Word`, `Markdown`, `Html`, `All`)
- `ProgressTracker` — tracks documentation progress across parallel tasks
- `CrossDocLinkHelper` — cross-document hyperlink resolution
- `ZipHelper`, `JsonUtil`, `YamlDotNet` — file parsing utilities

**NuGet dependencies** (all managed in `PowerDocu.Common.csproj`):
| Package | Version | Purpose |
|---|---|---|
| `DocumentFormat.OpenXml` | 3.5.1 | Word document generation |
| `Grynwald.MarkdownGenerator` | 3.0.106 | Markdown file creation |
| `Newtonsoft.Json` | 13.0.4 | JSON parsing |
| `Rubjerg.Graphviz` | 3.0.4 | Flow/app diagram generation (requires Graphviz) |
| `Svg` | 3.4.7 | SVG to PNG conversion |
| `HtmlAgilityPack` | 12.4 | HTML parsing |
| `System.Drawing.Common` | 10.0.5 | Drawing utilities |
| `Microsoft.PowerFx.Core` | 1.8.1 | Power Fx expression parsing |
| `YamlDotNet` | 16.3.0 | YAML parsing (agent definitions) |

---

## Project Dependency Graph

```
PowerDocu.GUI (WinExe, net10.0-windows)
  └── PowerDocu.FlowDocumenter
  └── PowerDocu.AppDocumenter
  └── PowerDocu.SolutionDocumenter
        └── PowerDocu.FlowDocumenter
        └── PowerDocu.AppDocumenter
        └── PowerDocu.AgentDocumenter
        └── PowerDocu.AppModuleDocumenter
        └── PowerDocu.AIModelDocumenter
        └── PowerDocu.BPFDocumenter
        └── PowerDocu.DesktopFlowDocumenter

All documenter projects → PowerDocu.Common (submodule)
```

`PowerDocu.GUI` references only the three top-level documenters directly. `SolutionDocumenter` acts as the full orchestrator when processing `.zip` solution packages.

---

## Architecture: Two-Phase Documentation Pipeline

When processing a `.zip` solution package, `SolutionDocumentationGenerator` implements a two-phase pipeline:

### Phase 1 — Parse (collect all entities)
1. `FlowDocumentationGenerator.ParseFlows()` → `List<FlowEntity>`
2. `AppDocumentationGenerator.ParseApps()` → `List<AppEntity>`
3. `AgentDocumentationGenerator.ParseAgents()` → `List<AgentEntity>`
4. `SolutionParser` → `SolutionEntity` + `CustomizationsEntity` (tables, roles, BPFs, desktop flows, AI models, app modules)
5. All results stored in a shared `DocumentationContext`

### Phase 2 — Generate output (write docs)
Generators are called in order with the shared `DocumentationContext`:
1. `FlowDocumentationGenerator.GenerateOutput()` — parallel per flow
2. `AppDocumentationGenerator.GenerateOutput()` — parallel per app
3. `AgentDocumentationGenerator.GenerateOutput()` — sequential per agent
4. `AIModelDocumentationGenerator.GenerateOutput()`
5. `BPFDocumentationGenerator.GenerateOutput()`
6. `DesktopFlowDocumentationGenerator.GenerateOutput()`
7. `AppModuleDocumentationGenerator.GenerateOutput()`
8. `SolutionDocumentationContent` + graph builders + `SolutionWordDocBuilder` / `SolutionMarkdownBuilder` / `SolutionHtmlBuilder`

**Standalone `.msapp` files** bypass the orchestrator and go directly to `AppDocumentationGenerator.GenerateDocumentation()`.

---

## Naming Conventions

### File/Class naming pattern (consistent across all documenters)

Each documenter project follows the same naming pattern:

| File | Purpose |
|---|---|
| `<X>DocumentationGenerator.cs` | Static entry point: `ParseX()`, `GenerateOutput()`, `GenerateDocumentation()` (legacy) |
| `<X>DocumentationContent.cs` | Builds the content model (sections, tables, text) from the entity |
| `<X>WordDocBuilder.cs` | Extends `WordDocBuilder`; writes `.docx` |
| `<X>MarkdownBuilder.cs` | Extends `MarkdownBuilder`; writes `.md` files |
| `<X>HtmlBuilder.cs` | Extends `HtmlBuilder`; writes `.html` files |
| `GraphBuilder.cs` | Creates Graphviz diagrams (PNG + SVG) |

### Namespace pattern
`PowerDocu.<ComponentName>` — e.g., `PowerDocu.FlowDocumenter`, `PowerDocu.AppDocumenter`

### Output folder naming
Output is written relative to the source file location:
- Solution: `Solution <SafeName>\`
- Flows: `Solution <SafeName>\FlowDoc <SafeName>\`
- Apps: `Solution <SafeName>\AppDoc <SafeName>\`
- Agents: `Solution <SafeName>\AgentDoc <SafeName>\`
- Model-Driven Apps: inline in the solution folder

`CharsetHelper.GetSafeName()` sanitizes names to be filesystem-safe.

---

## DocumentationContext

`DocumentationContext` (in `PowerDocu.Common`) is the central shared state object passed through the pipeline:

```csharp
public class DocumentationContext
{
    public List<FlowEntity> Flows;
    public List<AppEntity> Apps;
    public List<AgentEntity> Agents;
    public List<AppModuleEntity> AppModules;
    public List<BPFEntity> BusinessProcessFlows;
    public List<DesktopFlowEntity> DesktopFlows;
    public List<TableEntity> Tables;
    public List<SecurityRole> Roles;
    public SolutionEntity Solution;
    public CustomizationsEntity Customizations;
    public ConfigHelper Config;
    public bool FullDocumentation;
    public string OutputPath;
    public string SourceZipPath;
    public ProgressTracker Progress;
}
```

When adding support for a new component type, add its collection to `DocumentationContext`.

---

## Adding a New Documenter

To add documentation for a new Power Platform component type (e.g., `Foo`):

1. **Create project** `PowerDocu.FooDocumenter/` with a `.csproj` referencing `PowerDocu.Common`
2. **Add entity** `FooEntity.cs` to `PowerDocu.Common`
3. **Add parser** `FooParser.cs` to `PowerDocu.Common`
4. **Add collection** `List<FooEntity> Foos` to `DocumentationContext`
5. **Implement** `FooDocumentationContent.cs`, `FooWordDocBuilder.cs`, `FooMarkdownBuilder.cs`, `FooHtmlBuilder.cs`, `FooDocumentationGenerator.cs`
6. **Add config flag** `documentFoos` to `ConfigHelper` and wire to `CommandLineOptions`
7. **Reference** the new project from `PowerDocu.SolutionDocumenter.csproj`
8. **Integrate** parse call in `SolutionDocumentationGenerator` Phase 1, generate call in Phase 2
9. **Add to solution** `PowerDocu.sln`

---

## Build & Development

### Prerequisites (Windows)
- .NET 10 SDK
- Git (for submodule management)
- Visual Studio Code with C# extension (or Visual Studio 2022+)

### First-time setup
```bash
git clone https://github.com/modery/PowerDocu
cd PowerDocu
git submodule init
git submodule update
```

### Build commands
```bash
# Debug build
dotnet build PowerDocu.GUI/PowerDocu.GUI.csproj

# Release build (framework-dependent, Windows x64)
dotnet clean
dotnet publish -c Release -r win-x64 /p:SelfContained=false

# Standalone build (includes .NET runtime)
dotnet publish -c Release -r win-x64 /p:SelfContained=true
```

### VS Code
- **F5** launches `PowerDocu GUI` debug config (builds then runs `bin/Debug/net10.0-windows/PowerDocu.exe`)
- Build task: `Ctrl+Shift+B` → "build"
- Publish task: available via Terminal → Run Task → "publish"

### Assembly output
The GUI project is a `WinExe` targeting `net10.0-windows` with `UseWindowsForms=true`. Assembly name is `PowerDocu` (not `PowerDocu.GUI`).

Post-build targets copy `glib-2.dll` and `gobject-2.dll` from `lib/` to the output root (updated versions to patch Graphviz library vulnerabilities).

---

## Configuration

Settings are stored at `%APPDATA%\PowerDocu\powerdocu.config.json`. The CLI does **not** load the saved config file — all CLI runs use defaults unless overridden by flags.

Key `ConfigHelper` properties:
- `outputFormat` — `"Word"`, `"Markdown"`, `"Html"`, or `"All"` (default: `"All"`)
- `documentChangesOnlyCanvasApps` — only document modified Canvas App properties (default: `true`)
- `documentDefaultValuesCanvasApps` — include default property values (default: `true`)
- `flowActionSortOrder` — `"By name"` or `"By order of appearance"` (default: `"By name"`)
- `wordTemplate` — path to a `.docx`/`.docm`/`.dotx` template
- `addTableOfContents` — add TOC to Word docs (default: `false`)
- `documentSolution`, `documentFlows`, `documentApps`, `documentAgents`, `documentModelDrivenApps`, `documentBusinessProcessFlows`, `documentDesktopFlows` — toggle each component type

---

## CLI Usage

```bash
PowerDocu.exe -q "path/to/solution.zip" -w -m -h
PowerDocu.exe -q "path/to/app.msapp" -w -t "path/to/template.docx"
PowerDocu.exe -i    # Update connector icons
```

Key flags: `-q` (items), `-w`/`-m`/`-h` (Word/Markdown/HTML), `-f` (full documentation), `-o` (output path), `-t` (Word template). See `settings.md` for the full reference.

---

## Notification System

All status and progress messages flow through `NotificationHelper`:
- `NotificationHelper.SendNotification(string)` — general log message
- `NotificationHelper.SendPhaseUpdate(string)` — "Parsing" / "Documenting" phase label
- `NotificationHelper.SendStatusUpdate(string)` — progress bar content

The GUI registers a UI notification receiver; the CLI registers a `ConsoleNotificationReceiver`. To add logging to a new context, implement and register a notification receiver via `NotificationHelper.AddNotificationReceiver()`.

---

## Graph Generation

Diagrams (flow charts, screen navigation graphs, agent topic graphs, site maps) are generated using **Graphviz** via the `Rubjerg.Graphviz` NuGet package. Each documenter that produces graphs has a `GraphBuilder.cs`. Output is both `.png` (via `ToPngFile`) and `.svg` (via `ToSvgFile`) in the component's output folder.

---

## Key Conventions

- **Static generators**: All `*DocumentationGenerator` classes are `static`.
- **Two-step generators**: Each generator exposes `ParseX()` + `GenerateOutput(context, path)` as the preferred API, plus a legacy `GenerateDocumentation()` method that does both in one call. Use the two-step API when integrating with `SolutionDocumenter`.
- **Parallel processing**: Flows and Apps use `Parallel.ForEach` with per-item output locks to prevent file collisions. Agents are processed sequentially (graph state is not thread-safe).
- **Word template**: If `config.wordTemplate` is null or the file doesn't exist, builders fall back to no template. Template is never modified in place.
- **Progress tracking**: Call `context.Progress?.Increment("ComponentTypeName")` at the end of each item's processing. Register all component counts before Phase 2 begins.
- **Output paths**: Always use `CharsetHelper.GetSafeName()` before appending user-provided names to filesystem paths.
- **`FullDocumentation` flag**: When `false`, only diagrams/graphs are generated (no Word/Markdown/HTML). This mode is used when the user clicks "Next" without toggling full docs.
- **Language version**: All projects use `<LangVersion>latest</LangVersion>`.

---

## Common Pitfalls

- The submodule at `modules/PowerDocu.Common` must be initialized before building. If you get "project not found" errors, run `git submodule init && git submodule update`.
- `PowerDocu.Common` changes must be committed in the submodule repo separately, then the parent repo's submodule pointer updated.
- The `glib-2.dll` / `gobject-2.dll` files in `PowerDocu.GUI/lib/` are required at runtime for Graphviz. They are automatically copied to the output directory by post-build MSBuild targets.
- The CLI does not load `powerdocu.config.json` — defaults always apply unless flags are passed.
- When the Word template path is specified but invalid/missing, the builders silently fall back to no template (no exception is thrown).

---
> Source: [modery/PowerDocu](https://github.com/modery/PowerDocu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
