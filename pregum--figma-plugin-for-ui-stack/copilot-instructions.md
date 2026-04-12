## figma-plugin-for-ui-stack

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Figma plugin project for managing UI component states. The plugin helps designers create and validate components with 5 standard states: default, hover, active, disabled, and loading.

## Development Commands

### Building the Plugin
```bash
npm run build              # Build for production
npm run build:watch        # Watch mode for development
```

### Linting
```bash
npm run lint              # Run ESLint on TypeScript files
```

### Testing the Plugin
1. Open Figma Desktop App
2. Navigate to **Plugins** > **Development** > **[Plugin Name]**
3. The plugin will load with any recent changes (when using watch mode)

## Project Structure

The plugin follows the standard Figma plugin architecture:

- **src/main.ts** - Main plugin logic running in the Figma sandbox environment (Node-like context)
- **src/ui.html** - User interface HTML
- **src/ui.ts** - UI logic running in browser context
- **manifest.json** - Plugin configuration and metadata
- **webpack.config.js** - Build configuration
- **dist/** - Compiled output (referenced in manifest.json)

## Architecture

### Two-Process Model

Figma plugins run in two separate contexts:

1. **Main Thread (main.ts)**:
   - Runs in a sandboxed Node-like environment
   - Has access to Figma Plugin API (`figma` object)
   - Can read/modify the Figma document
   - Cannot access DOM or browser APIs

2. **UI Thread (ui.html/ui.ts)**:
   - Runs in a browser iframe context
   - Can use DOM, CSS, and browser APIs
   - Cannot directly access Figma Plugin API
   - Communicates with main thread via `postMessage`

### Communication Pattern

```typescript
// Main thread sends to UI
figma.ui.postMessage({ type: 'action', data: value });

// UI sends to main thread
parent.postMessage({ pluginMessage: { type: 'action', data: value } }, '*');

// Main thread receives from UI
figma.ui.onmessage = msg => {
  if (msg.type === 'action') {
    // Handle message
  }
};
```

## State Management System

The plugin implements a 5-state system for UI components:

- **DEFAULT**: Base component state
- **HOVER**: Mouse hover interaction
- **ACTIVE**: Active/pressed state
- **DISABLED**: Non-interactive state
- **LOADING**: Loading/processing state

Components are organized as frames with naming convention: `ComponentName/state`

## Key Figma API Patterns

### Creating Nodes
```typescript
const frame = figma.createFrame();
const rect = figma.createRectangle();
figma.currentPage.appendChild(frame);
```

### Accessing Selection
```typescript
const selection = figma.currentPage.selection;
```

### Async Font Loading
```typescript
await figma.loadFontAsync({ family: "Inter", style: "Regular" });
```

### Showing UI
```typescript
figma.showUI(__html__, { width: 320, height: 240 });
```

## TypeScript Configuration

The project uses TypeScript with Figma plugin typings (`@figma/plugin-typings`). Type checking is enforced during the build process through webpack and can be validated with the lint command.

## Development Notes

- Always use `figma.loadFontAsync()` before modifying text nodes
- The `__html__` variable in main.ts is automatically injected by webpack from ui.html
- Use `figma.notify()` to show toast messages to users
- Call `figma.closePlugin()` when the plugin's task is complete
- Use batch operations when modifying multiple nodes for better performance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pregum)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pregum)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
