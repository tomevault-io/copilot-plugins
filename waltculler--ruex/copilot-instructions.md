## ruex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Tauri desktop application built with Next.js and TypeScript. The project combines a React frontend with a Rust backend to create a cross-platform desktop app. The application title is "ruex - for extra images" and is categorized as a developer tool.

## Architecture
- **Frontend**: Next.js 13 with TypeScript, React 18, and Sass for styling
- **Backend**: Tauri 1.2+ with Rust for native desktop functionality
- **Build System**: Next.js builds the frontend, Tauri handles the desktop packaging
- **Communication**: Frontend communicates with Rust backend through Tauri's invoke API

## Key Directories
```
src/                    # Next.js frontend source
├── components/         # React components (Card, CardButton)
├── hooks/             # Custom React hooks
│   └── tauri/         # Tauri-specific hooks (shortcuts.ts)
├── pages/             # Next.js pages
└── styles/            # SCSS stylesheets

src-tauri/             # Rust backend source
├── src/               # Rust source code
│   └── main.rs        # Main Tauri application entry point
├── Cargo.toml         # Rust dependencies and configuration
├── tauri.conf.json    # Tauri application configuration
└── icons/             # Application icons for various platforms
```

## Development Commands

### Frontend Development
- `npm run dev` - Start Next.js development server on http://localhost:3000
- `npm run build` - Build Next.js for production
- `npm run export` - Export static Next.js build
- `npm run start` - Start production Next.js server
- `npm run lint` - Run ESLint on the codebase

### Tauri Desktop App
- `npm run tauri dev` - Start Tauri development with live reload
- `npm run tauri build` - Build production desktop app for current platform
- `npm run tauri` - Access Tauri CLI directly

### Rust Backend
Navigate to `src-tauri/` directory for Rust-specific commands:
- `cargo run` - Run Rust backend directly (use Tauri commands instead)
- `cargo build` - Build Rust backend
- `cargo test` - Run Rust tests

## Tauri Configuration
- **Dev Path**: Frontend runs on http://localhost:3000 during development  
- **Build Commands**: Automatically runs `npm run build && npm run export` before building desktop app
- **Output Directory**: Frontend build outputs to `out/` directory for Tauri packaging
- **Window Settings**: 800x600 default size, resizable, titled "ruex - for extra images"

## Frontend-Backend Communication
The project uses Tauri's invoke system for frontend-backend communication:
- Frontend calls `invoke("command_name")` from `@tauri-apps/api/tauri`
- Rust backend defines commands with `#[tauri::command]` attribute
- Example: `on_button_clicked` command returns timestamp from Rust backend

## Custom Hooks
- `useGlobalShortcut` in `src/hooks/tauri/shortcuts.ts` - Registers global keyboard shortcuts

## Testing
No test framework is currently configured. Consider adding Jest/React Testing Library for frontend tests and standard Rust testing for backend.

## Platform Support
Configured to build for all platforms (Windows, macOS, Linux) with appropriate icons and bundles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WaltCuller) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
