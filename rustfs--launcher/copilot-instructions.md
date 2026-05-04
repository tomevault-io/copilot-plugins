## launcher

> **RustFS Launcher** is a desktop GUI application designed to configure, launch, and monitor the `rustfs` binary. It bridges a robust Rust backend with a reactive web-based frontend using the **Tauri** framework.

# Gemini Context: RustFS Launcher

## Project Overview
**RustFS Launcher** is a desktop GUI application designed to configure, launch, and monitor the `rustfs` binary. It bridges a robust Rust backend with a reactive web-based frontend using the **Tauri** framework.

- **Frontend:** Built with **Leptos** (Rust web framework), providing a type-safe, reactive UI for user input and real-time log streaming.
- **Backend:** A **Tauri** (Rust) application that manages the lifecycle of the external `rustfs` process, captures its stdout/stderr, and streams these logs to the frontend via events.

## Architecture & Key Components

### Backend (`src-tauri/`)
The backend acts as the process manager and native layer.
*   **`src-tauri/src/lib.rs`**: Application entry point. Sets up the Tauri builder, lifecycle hooks, and registers commands.
*   **`src-tauri/src/commands.rs`**: Defines the API exposed to the frontend (e.g., `launch_rustfs`, `get_app_logs`).
*   **`src-tauri/src/process.rs`**: Core logic for locating the `rustfs` binary (checks multiple paths), diagnosing it, and spawning the child process. Captures output streams.
*   **`src-tauri/src/state.rs`**: Manages global, thread-safe application state (process handles, log history) and emits real-time events (`app-log`, `rustfs-log`) to the frontend.

### Frontend (`src/`)
The frontend is a Single Page Application (SPA) compiled to WebAssembly.
*   **`src/main.rs`**: Leptos entry point, mounts the application.
*   **`src/app.rs`**: The main UI component. It handles:
    *   **Configuration:** Inputs for Host, Port, and Mountpoint.
    *   **Commands:** Invokes `launch_rustfs` via Tauri's `invoke` system.
    *   **Events:** Listens for `APP_LOG_EVENT` and `RUSTFS_LOG_EVENT` to update the log console in real-time.
*   **`src/logs.css`**: Styles specific to the log viewer component.

## Building and Running

### Prerequisites
*   **Rust**: Standard toolchain (`cargo`, `rustup`).
*   **Node.js**: Required for Tauri's CLI and frontend asset bundling.
*   **Trunk**: The WASM web application bundler (`cargo install trunk`).

### Setup & Execution
1.  **Download Binaries**: The application requires a platform-specific `rustfs` binary.
    *   **macOS/Linux**: Run `./build.sh`
    *   **Windows**: Run `build.bat`
    *   *Note: Binaries are placed in `src-tauri/binaries/` and are git-ignored.*

2.  **Development Server**:
    ```bash
    cargo tauri dev
    ```
    This starts the Tauri application with hot-reloading for both the Rust backend and the Leptos frontend.

3.  **Production Build**:
    ```bash
    cargo tauri build
    ```

## Development Conventions

### Code Style
*   **Rust**: Follow standard Rust idioms. Use `cargo fmt` and `cargo clippy --workspace --all-targets` before committing.
*   **Commits**: Adhere to **Conventional Commits** (e.g., `feat: add log filter`, `fix: process termination`).
*   **Organization**:
    *   Keep frontend code in `src/`.
    *   Keep backend logic in `src-tauri/src/`.
    *   Tests should be co-located with code in `#[cfg(test)]` modules.

### Testing
*   **Backend**: Run `cargo test -p rustfs-launcher`.
*   **Integration**: Manually verify the "Launch" flow and log streaming. Ensure the child process terminates when the window closes.

### File Structure Highlights
*   `Trunk.toml`: Configuration for the Leptos build pipeline.
*   `AGENTS.md`: Detailed repository guidelines for contributors and AI agents.

## Communication Guidelines
*   **Language**: All communication with the user must be in Chinese.

---
> Source: [rustfs/launcher](https://github.com/rustfs/launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
