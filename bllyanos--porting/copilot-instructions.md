## porting

> **Porting** is a Terminal User Interface (TUI) application for managing SSH port forwarding, built using [OpenTUI](https://github.com/opentui/opentui) and [Solid.js](https://www.solidjs.com/). It leverages [Bun](https://bun.sh/) as its runtime, package manager, and test runner. The project provides a reactive development experience for building terminal-based applications using familiar web-like primitives (JSX, components, signals).

# GEMINI.md - Context for Porting Project

## Project Overview
**Porting** is a Terminal User Interface (TUI) application for managing SSH port forwarding, built using [OpenTUI](https://github.com/opentui/opentui) and [Solid.js](https://www.solidjs.com/). It leverages [Bun](https://bun.sh/) as its runtime, package manager, and test runner. The project provides a reactive development experience for building terminal-based applications using familiar web-like primitives (JSX, components, signals).

### Main Technologies
- **Runtime & Tooling:** Bun
- **UI Framework:** Solid.js
- **TUI Renderer:** OpenTUI (@opentui/core, @opentui/solid)
- **Language:** TypeScript

### Architecture
The application follows a reactive architecture where the UI is declared using JSX components provided by `@opentui/solid`. The entry point is `src/index.tsx`, which uses the `render` function from `@opentui/solid` to mount the application.

## Building and Running

### Prerequisites
- [Bun](https://bun.sh/) installed on your system.

### Commands
- **Install Dependencies:**
  ```bash
  bun install
  ```
- **Run in Development Mode (with hot-reload/watch):**
  ```bash
  bun dev
  ```
  *Note: This executes `bun run --watch src/index.tsx`.*
- **Type Checking:**
  ```bash
  bunx tsc --noEmit
  ```

## Development Conventions

### Code Style & Structure
- **TypeScript:** The project is strictly typed using TypeScript. Always provide types for component props and logic.
- **JSX/TSX:** Use `.tsx` for files containing UI components. The project uses the Solid.js JSX transform with `@opentui/solid` as the import source.
- **Components:** Organize the UI into reusable Solid.js components.
- **Reactivity:** Utilize Solid.js primitives like `createSignal`, `createMemo`, and `createEffect` for state management.

### Configuration
- **package.json:** Contains scripts and dependencies.
- **tsconfig.json:** Configured for Bun and OpenTUI (note `jsxImportSource: "@opentui/solid"`).
- **bunfig.toml:** Configuration for the Bun runtime.

### TUI Primitives
Use OpenTUI components like `<box>`, `<text>`, `<ascii_font>`, etc., to build the interface. Refer to OpenTUI documentation for available props (e.g., `alignItems`, `justifyContent`, `flexGrow`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bllyanos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
