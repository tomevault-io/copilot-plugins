## mini-transfer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**迷你传输助手** (Mini Transfer Assistant) - A lightweight cross-platform desktop application for file transfer between PC and mobile devices.

**Tech Stack:**
- **Frontend:** Vanilla TypeScript + Vite
- **Backend:** Rust + Tauri 2.x
- **HTTP Server:** Hyper (Rust)
- **QR Code:** qrcode crate

## Prerequisites

1. **Node.js** (for frontend build)
2. **Rust** (via rustup) - Required for Tauri backend
3. **System WebView** - Tauri uses system webview (no bundled Chromium)

## Install Rust (Windows)

Option 1 - Download installer:
- Visit https://rustup.rs/
- Download `rustup-init.exe`
- Run with default options

Option 2 - Using winget:
```bash
winget install Rustlang.Rustup
```

After installation, restart your terminal and verify:
```bash
cargo --version
```

## Common Commands

### Development
```bash
npm run dev          # Start dev server (frontend only)
npm run tauri dev    # Start full Tauri dev environment
```

### Building
```bash
npm run build        # Build frontend only
npm run tauri build  # Build complete application
```

### Build outputs (Windows)
- `src-tauri/target/release/` - Raw executable
- `src-tauri/target/release/bundle/` - Installer packages (msi, nsis)

## Project Structure

```
src-tauri/
├── src/
│   └── main.rs           # Rust backend - HTTP server, QR code, file operations
├── mobile/
│   └── index.html        # Mobile web interface (served by HTTP)
├── Cargo.toml            # Rust dependencies
├── tauri.conf.json       # Tauri configuration
└── build.rs              # Build script

src/
├── main.ts               # Frontend entry point
└── style.css             # Application styles

index.html                # Desktop app UI
vite.config.ts            # Vite configuration
```

## Key Architecture

### Frontend-Backend Communication
- Tauri `invoke()` calls from TypeScript to Rust commands
- Commands: `start_server`, `stop_server`, `generate_qr_code`

### HTTP Server (Rust)
- Runs on available port 8000-8999
- Endpoints:
  - `GET /` - Mobile web interface
  - `GET /api/files` - List files in shared folder
  - `GET /api/download?file=name` - Download file
  - `POST /api/upload` - Upload file (multipart)
- CORS enabled for mobile access

### File Upload Flow
- Mobile uploads → `POST /api/upload`
- Files saved to: `{shared_folder}/来自手机/`

## Window Configuration

- Size: 400×500 (fixed, non-resizable)
- Centered on screen
- Title: "迷你传输助手"

## Reducing Build Size

To keep the application under 10MB:

1. **Use `cargo strip`** (Windows):
   ```bash
   cd src-tauri
   cargo build --release
   strip target/release/mini-transfer.exe
   ```

2. **Enable LTO** in `Cargo.toml`:
   ```toml
   [profile.release]
   lto = true
   codegen-units = 1
   opt-level = "z"
   strip = true
   ```

---
> Source: [kavil/mini-transfer](https://github.com/kavil/mini-transfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
