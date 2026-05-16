## rae

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rae is a desktop AI assistant built with Tauri v2 that provides an overlay interface (the "magic dot") that can be summoned anywhere on your screen via global shortcuts. The app combines a React/TypeScript frontend with a Rust backend for native system integrations.

## Development Commands

### Setup
```bash
npm install
```

### Development
```bash
npm run tauri dev
```
This starts both the Vite dev server (port 5174) and the Tauri app.

### Build
```bash
npm run build        # Builds frontend (TypeScript + Vite)
npm run tauri build  # Builds full Tauri application
```

### Frontend Only
```bash
npm run dev      # Run Vite dev server only
npm run preview  # Preview production build
```

## Architecture

### Dual-Window System

The app uses **two main windows** managed by Tauri:

1. **Main Window** (`main`): Full-featured app with chat, settings, onboarding, agents, and notes
2. **Overlay Window** (`overlay`): The "magic dot" - a draggable, always-on-top overlay that follows the mouse and provides quick AI access

Both windows hide to system tray on close instead of exiting the app.

### Frontend Architecture (React + TypeScript)

- **Routing**: React Router v7 with nested routes
  - Root: `/` (Onboarding)
  - Main app: `/app/*` (Landing, Chat, Agents, Notes, Settings)
  - Overlay: `/overlay` (Magic dot interface)

- **State Management**:
  - **Zustand stores** in `src/store/`:
    - `chatStore.ts`: Chat messages, conversation history, overlay chat state
    - `userStore.ts`: User authentication and profile
    - `darkThemeStore.ts`: Theme preferences
    - `noteStore.ts`: Notes/brain functionality
    - `appStore.ts`: General app state
  - **Jotai**: Used with Provider for atomic state in overlay components

- **Path Alias**: `@/*` maps to `src/*` (configured in vite.config.ts and tsconfig.json)

### Backend Architecture (Rust + Tauri)

- **Main entry**: `src-tauri/src/main.rs`
  - Sets up system tray
  - Registers global shortcuts
  - Manages window lifecycle
  - Intercepts close events to hide windows instead of exiting

- **Module structure** (`src-tauri/src/`):
  - `functions/`: Tauri commands organized by feature
    - `overlay.rs`: Magic dot positioning, following, pinning, toggling
    - `chat.rs`: Clipboard/selection watchers, @rae mention detection
    - `stealth.rs`: Stealth mode for hiding from screen capture
    - `supermemory.rs`: Memory/brain backend integration
    - `app.rs`: App-level commands
  - `services/`: System-level services
    - `window_capture.rs`: Window screenshot capture
    - `text_injection.rs`: Text injection into other windows
    - `screenshot_service.rs`: Screenshot functionality
    - `auto_start.rs`: Launch on startup
  - `platform.rs`: Platform-specific utilities
  - `utils.rs`: Smooth animations, monitor detection
  - `audio_client.rs`: Audio input handling

### Key Features

1. **Global Shortcuts** (registered with 2-4 second startup delays):
   - `Ctrl+H`: Toggle magic dot visibility
   - `Ctrl+M`: Center overlay bar
   - `Ctrl+P`: Toggle pin overlay

2. **Auto-show Triggers**:
   - Clipboard copy detection
   - Text selection detection
   - `@rae` mention detection (listens for typing "@rae" in any app)

3. **Overlay Behaviors**:
   - Follows cursor when unpinned
   - Pins to top-center of screen
   - Notch mode for MacBook-style display
   - Smooth animations for movement and resizing

4. **Window Management**:
   - Transparent, frameless, always-on-top windows
   - Window vibrancy effects
   - Skip taskbar when hidden
   - System tray integration with show/quit menu

### API Integration

- API calls in `src/api/`:
  - `auth.ts`: User authentication
  - `chat.ts`: Conversation management (`GetConvos`, etc.)
  - `notes.ts`: Notes/brain operations
  - `fetch.ts`: Base fetch utilities
  - `updates.ts`: App update checks

### Important Configuration

- **Vite config**: Frontend dev server runs on port 5174 (strict port)
- **TypeScript**: Strict mode disabled, allows some loose typing
- **Tauri config**: `src-tauri/tauri.conf.json`
  - Defines window properties (transparent, decorations: false, etc.)
  - Security capabilities for different windows
  - Plugin configurations (shell, autostart)

## Development Notes

### When Working with the Overlay

- The overlay window is created dynamically via `WebviewWindowBuilder` in Rust
- Position/size changes should use smooth animations via `utils::smooth_move` and `utils::smooth_resize`
- The overlay state is managed both in Rust (position, visibility) and React (UI state)
- Monitor detection handles multi-monitor setups

### When Working with Global Shortcuts

- Shortcuts are registered with cooldowns to prevent rapid firing
- Registration is delayed 2-4 seconds after startup to improve performance
- Always check `isRegistered` and `unregister` before registering to avoid conflicts
- Cleanup shortcuts in React `useEffect` return functions

### When Working with Window Management

- Windows never truly close - they hide and set `skip_taskbar: true`
- The main window intercepts `CloseRequested` events in Rust
- Use `invoke("show_magic_dot")` or `invoke("toggle_magic_dot")` from frontend
- System tray provides fallback access when all windows are hidden

### When Adding New Tauri Commands

1. Add function to appropriate module in `src-tauri/src/functions/`
2. Add to `invoke_handler!` macro in `src-tauri/src/main.rs`
3. Call from frontend using `invoke("command_name", { args })`
4. Add permissions to appropriate capability file if needed

### Windows-Specific Code

The codebase includes extensive Windows-specific integrations:
- UI Automation for text selection detection
- Window handle (HWND) manipulation
- Screenshot capture of specific windows
- Text injection using Enigo

These are all Windows-only and use `winapi` and `windows` crates.

---
> Source: [rae-app/rae](https://github.com/rae-app/rae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
