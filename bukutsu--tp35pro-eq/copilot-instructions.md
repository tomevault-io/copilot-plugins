## tp35pro-eq

> - **USB Communication**: This app uses **Tauri v2** with a custom Rust backend for direct USB communication with the TP35 Pro DAC via `rusb`.

# Agent Instructions for TP35 Pro EQ

## Architecture & Communication

- **USB Communication**: This app uses **Tauri v2** with a custom Rust backend for direct USB communication with the TP35 Pro DAC via `rusb`.
- **Tauri Role**: Tauri handles the main process, window management, and native bridge for hardware operations. USB logic lives in `src-tauri/src/`.
- **Auto-Connect**: The app automatically connects to the TP35 Pro device on startup.

## Core Commands

- `npm run dev`: Starts Vite dev server and Tauri development window.
- `npm run build`: Packages the app for the current platform.
- `npm run tauri build`: Full production build of the Tauri bundle.

IMPORTANT: when running long-lived dev commands from an automated agent (eg. `npm run dev`, `tauri dev`, or `cargo run` without exit), always use an explicit timeout and avoid leaving the process attached indefinitely. Agents should prefer running short-lived test binaries (or use the Bash tool with the `timeout` parameter) so they don't block further automated actions or log capture.

## Git Versioning

- **Branching**: Use `main` for production, create feature branches like `feature/your-feature-name`.
- **Commit Messages**: Follow Conventional Commits format:
  - `feat: add new feature` - for new features
  - `fix: fix bug` - for bug fixes
  - `chore: update dependencies` - for maintenance
  - `docs: update documentation` - for docs
- **Releases**: Use git tags for versioning:
  - `git tag v1.0.0` - for releases
  - `git push origin main --tags` - to push tags
- **GitHub Remote**: The remote is set to `git@github.com:MCPC/tp35pro-eq.git`

## Development Gotchas

- **Rust Backend**: All PEQ codec logic and USB HID communication is now handled in the Rust core (`src-tauri`).
- **Persistence**: User profiles are managed via the Tauri state and stored in the application data directory.
- **Native Dependencies**: Requires Rust (Cargo) and `libusb` headers on Linux.

## UI Standards

- **Tailwind CSS**: The v2 rewrite uses Tailwind for styling.
- **Responsiveness**: The app uses a responsive grid layout. Keep components modular in `src/components/`.

## Build Requirements

- **Rust Toolchain**: Must have `rustc` and `cargo` installed.
- **Linux**: Requires `libusb` development headers (`libusb-1.0-0-dev`) and standard build-essential.
- **Windows**: Requires WebView2 and Microsoft C++ Build Tools.
- **macOS**: Requires Xcode Command Line Tools.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bukutsu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
