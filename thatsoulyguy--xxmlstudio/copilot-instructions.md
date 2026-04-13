## xxmlstudio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Configure (run from project root)
cmake -B build -G Ninja

# Build
cmake --build build --config Debug
cmake --build build --config Release

# Clean and rebuild
cmake --build build --clean-first
```

The build output is `build/XXMLStudio.exe`. The bundled XXML toolchain is automatically copied to `build/toolchain/` post-build.

## Architecture Overview

XXMLStudio is a Qt 6 IDE for the XXML programming language. All code is in the `XXMLStudio` namespace.

### Core Components

**Application Singleton** (`src/core/Application.h`) - Bootstrap and lifecycle management. Creates MainWindow, Settings, manages themes. Entry: `Application::create()` → `run()` → `destroy()`.

**MainWindow** (`src/ui/MainWindow.h`) - Central orchestrator. Owns ProjectManager, BuildManager, LSPClient, BookmarkManager. Creates all dock widgets and coordinates builds, editor tabs, and project operations.

**Settings** (`src/core/Settings.h`) - QSettings wrapper with typed accessors and change signals for window state, editor preferences, toolchain paths.

### Editor System

**CodeEditor** (`src/editor/CodeEditor.h`) - QPlainTextEdit subclass with line numbers, current line highlight, syntax highlighting, diagnostic underlines, bracket matching, auto-closing pairs, find/replace.

**EditorTabWidget** (`src/editor/EditorTabWidget.h`) - Tab container managing open files with unsaved change tracking.

**XXMLSyntaxHighlighter** (`src/editor/XXMLSyntaxHighlighter.h`) - XXML syntax highlighting with three themes (Darcula, QtCreator, VSCodeDark). Uses FormatType enum for dynamic theme switching.

### LSP Integration

**LSPClient** (`src/lsp/LSPClient.h`) - High-level async LSP API. States: Disconnected → Connecting → Initializing → Ready. Provides completion, hover, goto definition, references, document symbols.

**JsonRpcClient** (`src/lsp/JsonRpcClient.h`) - Low-level JSON-RPC 2.0 over stdio. Manages LSP server process, Content-Length protocol, request/response matching.

**LSPProtocol** (`src/lsp/LSPProtocol.h`) - Header-only LSP data structures and JSON serialization.

### Build System

**BuildManager** (`src/build/BuildManager.h`) - Orchestrates build pipeline. States: Idle → ResolvingDependencies → Compiling → Linking → Finished. Coordinates DependencyManager → CompilerInvoker → OutputParser.

**CompilerInvoker** (`src/build/CompilerInvoker.h`) - Spawns XXML compiler process with async output streaming.

**DependencyManager** (`src/project/DependencyManager.h`) - Resolves Git-based dependencies with caching. Uses GitFetcher for clone/fetch operations.

### Project Management

**Project** (`src/project/Project.h`) - Represents .xxmlp project file. Contains build configs (Debug/Release), run configs, dependencies, include paths.

**ProjectManager** (`src/project/ProjectManager.h`) - Singleton managing current project lifecycle.

### Panels

All in `src/panels/`: ProjectExplorer (file tree), ProblemsPanel (errors/warnings table), BuildOutputPanel, TerminalPanel, OutlinePanel (LSP document symbols).

### Dialogs

All in `src/dialogs/`: FindReplaceDialog, GoToLineDialog, SettingsDialog, NewProjectDialog.

## Key Interaction Flows

**Build Flow**: MainWindow → BuildManager → DependencyManager (fetch deps) → CompilerInvoker (compile) → OutputParser (parse) → ProblemsPanel (display)

**LSP Flow**: CodeEditor changes → LSPClient::changeDocument() → JsonRpcClient → LSP server → diagnosticsReceived signal → CodeEditor::setDiagnostics() + ProblemsPanel

**Project Open**: MainWindow → ProjectManager::openProject() → Project::load() → ProjectExplorer::setRootPath() → LSPClient::setProjectRoot()

## Dependencies

- Qt 6.10.1 (Core, Gui, Widgets, Network, Concurrent)
- CMake 3.16+
- C++17 compiler (LLVM MinGW on Windows)
- Bundled: XXML compiler and LSP server in `toolchain/`

## Code Patterns

- Qt signal/slot for decoupled async communication
- QProcess for subprocess management (compiler, LSP server)
- All file paths use native separators via `QDir::toNativeSeparators()`
- Settings changes emit signals for live UI updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThatSoulyGuy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ThatSoulyGuy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
