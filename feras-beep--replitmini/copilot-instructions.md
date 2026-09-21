## replitmini

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Mini Replit** - a browser-based code editor and execution environment that runs entirely in the client. It's a single-page application supporting multiple programming modes with live preview capabilities.

## Development Commands

```bash
# Start development server (default: http://localhost:5173)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Architecture

### Core Structure
- **index.html** - Single HTML file containing all UI structure and CSS styles
- **main.js** - All application logic (no build-time bundling of JS modules)
- Uses **Vite** only as a dev server; the app itself has zero runtime dependencies

### Four Execution Modes

1. **JavaScript Mode** - Runs JS code in a Web Worker sandbox
2. **Python Mode** - Uses Pyodide (Python → WebAssembly) loaded from CDN
3. **Web App Mode** - Multi-file static site builder with virtual filesystem
4. **Node Mode** - Full Node.js environment via WebContainers API (requires COOP/COEP headers)

### Key Architectural Patterns

**Dynamic Dependency Loading**
- Ace Editor, Pyodide, and WebContainers are loaded dynamically from CDNs with fallback URLs
- Graceful degradation: if Ace fails, falls back to plain textarea editor
- See `Editor.tryLoadAce()`, `ensurePyodide()`, `ensureWebContainers()`

**Virtual Filesystem (VFS)**
- Web App and Node modes use a VFS stored in localStorage
- Each mode has separate storage keys: `mini-repl-vfs` (webapp) and `mini-repl-node-vfs` (node)
- VFS structure: `{ files: {[path]: content}, active: currentFilePath }`
- Node mode VFS is converted to WebContainer tree format via `vfsToTree()`

**Preview System**
- Web App mode: HTML inlined via `inlineAssets()` then rendered in sandboxed iframe using `srcdoc`
- Node mode: WebContainer spawns actual Node.js server, iframe connects to server URL
- Console bridging: Preview iframe sends console.log/warn/error via postMessage to main window

**Console Management**
- Global `ConsoleSettings` object tracks: timestamps, clearOnRun, maxLines (500), isExpanded state
- Console auto-expands when receiving output, can be manually toggled
- Line limiting prevents memory bloat by removing oldest lines

**Layout System**
- Main layout uses flexbox with resizable splitter between editor and preview panes
- Splitter position (30-70% range) persists to localStorage
- Console is collapsible with height constraints (40px minimized, 150-400px expanded)

## Important Implementation Details

**Code Execution Safety**
- JavaScript runs in Web Worker (isolated from main thread)
- Web App iframe has `sandbox="allow-scripts allow-forms allow-popups allow-modals"`
- Python runs in Pyodide sandbox

**Auto-save and Auto-refresh**
- 1-second interval checks for changes in project modes (webapp/node)
- Web App mode auto-refreshes preview on content change
- Node mode requires explicit restart for server changes

**Mode Switching**
- Saves current editor content to localStorage before switching
- Each mode (js/python/webapp/node) has separate localStorage keys
- Editor syntax highlighting mode changes based on file extension or mode

**WebContainers Limitation**
- Requires `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers
- Will show warning and gracefully skip if headers not present
- Check `self.crossOriginIsolated` to detect availability

## Modifying UI Layout

The layout is CSS-in-HTML with CSS custom properties in `:root` for theming. Key classes:
- `.main` - Flexbox container for editor/preview split
- `.workspace` - Editor pane with optional file bar
- `.right` - Preview pane with collapsible console
- `.console-container` - Toggle between `.minimized` and `.expanded` classes
- `.splitter` - Draggable divider between panes

## Testing

The app includes a self-test suite accessible via the "✓ Self-test" button that validates all four execution modes. No separate test framework is used.

---
> Source: [feras-beep/replitmini](https://github.com/feras-beep/replitmini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
