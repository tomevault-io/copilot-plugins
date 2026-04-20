## opencode-obsidian

> Guidelines for AI coding agents working on the obsidian-opencode plugin.

# AGENTS.md - Obsidian OpenCode Plugin

Guidelines for AI coding agents working on the obsidian-opencode plugin.

## Project Overview

Obsidian plugin that embeds the OpenCode AI assistant via an iframe. Spawns a local server process and displays its web UI in the Obsidian sidebar.

**Tech Stack:** TypeScript, Obsidian Plugin API, esbuild, Node.js child processes

## Build Commands

```bash
bun install          # Install dependencies
bun run build        # Production (type-check + bundle)
```

Output: `main.js` (CommonJS bundle)

## Project Structure

```
src/
├── main.ts           # Plugin entry, extends Plugin
├── types.ts          # Types and constants
├── OpenCodeView.ts   # Sidebar view (ItemView) with iframe
├── ProcessManager.ts # Server process lifecycle
└── SettingsTab.ts    # Settings UI (PluginSettingTab)
```

## Coding guidelines

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Classes | PascalCase | `OpenCodePlugin`, `ProcessManager` |
| Interfaces/Types | PascalCase | `OpenCodeSettings`, `ProcessState` |
| Constants | UPPER_CASE or camelCase | `DEFAULT_SETTINGS`, `OPENCODE_VIEW_TYPE` |
| Variables/functions | camelCase | `getVaultPath`, `startServer` |
| Private members | camelCase (no prefix) | `private processManager` |
| Files | PascalCase (classes), lowercase (entry) | `ProcessManager.ts`, `main.ts` |

### TypeScript Patterns
- `strictNullChecks` enabled - handle null/undefined
- Union types for state: `"stopped" | "starting" | "running" | "error"`
- `async/await` over Promises
- Explicit return types on public methods

```typescript
getProcessState(): ProcessState {
  return this.processManager?.getState() ?? "stopped";
}
```

### Obsidian API Patterns
- Extend `Plugin` with `onload()`/`onunload()` lifecycle
- Extend `ItemView` for views: `getViewType()`, `onOpen()`, `onClose()`
- Extend `PluginSettingTab` for settings: `display()`
- DOM helpers: `createEl()`, `createDiv()`, `setIcon()`
- Register in `onload()`, clean up in `onunload()`

```typescript
this.registerView(OPENCODE_VIEW_TYPE, (leaf) => new OpenCodeView(leaf, this));
this.addCommand({ id: "toggle-view", name: "Toggle panel", callback: () => this.toggleView() });
```

### DOM Creation
```typescript
const container = this.contentEl.createDiv({ cls: "opencode-container" });
container.createEl("h3", { text: "Title" });
container.createEl("button", { text: "Click", cls: "mod-cta" });
```

### State Management
- Callback-based subscriptions
- Centralized state in manager classes
- Immediate notification on state change

## Config Summary

**tsconfig.json:** ES6 target, ESNext modules, strictNullChecks, noImplicitAny

**esbuild:** CJS format, es2018 target, node platform. Externals: obsidian, electron, CodeMirror, Node builtins

## Desktop-Only

Uses Node.js APIs unavailable on mobile:
- `child_process.spawn()` for server process
- File system via vault adapter

Check for desktop environment before adding mobile-incompatible features.

---
> Source: [mtymek/opencode-obsidian](https://github.com/mtymek/opencode-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
