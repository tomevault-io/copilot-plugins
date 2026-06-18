## vscode-project-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A VS Code extension that adds a sidebar tree view for quickly opening projects discovered via directory scanning, organized into groups. Projects are automatically discovered by scanning configured directories for a marker file. Configuration (scan settings) is stored at `~/.config/vscode-project-manager/settings.json` (overridable via `projectManager.configFile`). Discovered projects are cached at `~/.cache/vscode-project-manager/projects.json`.

## Build Commands

```bash
bun run compile        # Compile TypeScript to out/
bun run watch          # Compile in watch mode
bun run package        # Package as .vsix using vsce
bash install.sh         # Build, install extension, and clean up .vsix
```

No test framework is configured. To test, use VS Code's Extension Development Host (F5 in VS Code).

## Architecture

5 source files in `src/`:

- **types.ts** - Data model: `Config` holds scan settings (`scanDirs`, `scanMarker`, `scanMaxDepth`). `Cache` holds `Group[]`, each `Group` contains `Project[]` (name + path).
- **configManager.ts** - Manages two files: config (scan settings) and cache (groups/projects). Provides `importScannedProjects()` for bulk-importing scan results and `removeStaleProjects()` for pruning entries whose paths no longer exist. Config path resolved via `getConfigPath()` which reads the `projectManager.configFile` VS Code setting. Cache path is `~/.cache/vscode-project-manager/projects.json`.
- **treeItems.ts** - VS Code `TreeItem` subclasses: `GroupItem` (collapsible folders) and `ProjectItem` (leaf nodes with click-to-open).
- **projectTreeProvider.ts** - `TreeDataProvider` that maps cached groups to the tree view. Two levels: groups at root, projects as children.
- **extension.ts** - Entry point. Registers command handlers, the tree provider, and a `FileSystemWatcher` to auto-reload when the config file changes externally. Contains the `scanProjects` walk logic that recursively finds projects by marker file. The refresh command automatically cleans stale projects and triggers a rescan.

## Key Conventions

- All commands are prefixed with `projectManager.` and registered in both `package.json` (contributes.commands + menus) and `extension.ts`
- When adding a new command: add it to `package.json` contributes.commands, add menu entries if needed (with appropriate `when` clauses), and register the handler in `extension.ts`
- Tree items use VS Code built-in `ThemeIcon`s (`folder`, `repo`, `refresh`, `gear`, `search`, `file-add`)
- The scan feature groups discovered projects by their top-level parent directory name within each scan root
- No manual project/group CRUD — all projects come from scanning

---
> Source: [kpkym/vscode-project-manager](https://github.com/kpkym/vscode-project-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
