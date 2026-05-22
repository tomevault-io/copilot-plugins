## pomodoro

> A dark-themed Pomodoro timer desktop application built with Electron and TypeScript.

# Pomodoro Timer - Electron Desktop App

A dark-themed Pomodoro timer desktop application built with Electron and TypeScript.

## Project Structure

```
src/
├── main/           # Electron main process
│   ├── index.ts        # App entry point
│   ├── window.ts       # Browser window management
│   ├── tray.ts         # System tray integration
│   ├── ipc-handlers.ts # IPC communication handlers
│   └── notification.ts # Native notifications
├── preload/        # Preload scripts (exposes safe APIs to renderer)
│   └── index.ts
├── renderer/       # Renderer process (UI)
│   ├── index.ts        # Entry point
│   ├── index.html      # HTML template
│   ├── components/     # UI components
│   │   ├── App.ts          # Main app orchestrator
│   │   ├── TimerDisplay.ts # Time display
│   │   ├── ProgressRing.ts # Circular progress indicator
│   │   ├── Controls.ts     # Start/Pause/Reset buttons
│   │   ├── SessionIndicator.ts # Work/Break indicator
│   │   └── SettingsPanel.ts    # Settings UI
│   ├── core/           # Business logic
│   │   ├── Timer.ts        # Timer engine
│   │   ├── PomodoroCycle.ts # Work/break cycle management
│   │   └── Settings.ts     # Settings persistence
│   ├── utils/
│   │   └── format.ts   # Time formatting utilities
│   └── styles/
│       ├── variables.css   # CSS custom properties
│       ├── base.css        # Base styles
│       └── components.css  # Component styles
└── shared/         # Types shared between processes
    ├── types.ts        # TypeScript interfaces
    └── constants.ts    # Default values
```

## Architecture

### Main Process
- Handles window creation, system tray, and native notifications
- IPC handlers expose safe APIs to renderer via preload script

### Renderer Process
- Pure TypeScript with no framework dependencies
- Component-based architecture with clear separation of concerns
- Timer class manages timing logic independently of UI

### Timer System
- `Timer.ts`: Core timing engine with tick/complete callbacks
- `PomodoroCycle.ts`: Manages work/break phase transitions
- Supports pause/resume with accurate elapsed time tracking

## Build Commands

```bash
npm run build       # Full build (main + renderer + assets)
npm run dev         # Build and run in Electron
```

## Key Features

- Work/break cycle with configurable durations
- System tray integration (app stays running when closed)
- Always-on-top option
- Native desktop notifications
- Settings persistence via localStorage
- Circular progress ring visualization

## Development Notes

- Uses esbuild for renderer bundling
- TypeScript compiles main process separately
- No hot reload; rebuild required for changes

---
> Source: [wjfengLove/Pomodoro](https://github.com/wjfengLove/Pomodoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
