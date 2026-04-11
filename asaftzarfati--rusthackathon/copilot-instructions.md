## rusthackathon

> This project is a full-stack Rust application designed as a **Simulation** component. It communicates with an external **Realtime** component via UDP using Protobuf-encoded messages. The application visualizes data and provides control via a web interface.

# Project Context: Rust Hackathon Simulation

## Overview
This project is a full-stack Rust application designed as a **Simulation** component. It communicates with an external **Realtime** component via UDP using Protobuf-encoded messages. The application visualizes data and provides control via a web interface.

## Architecture
The project is organized as a Cargo workspace with the following members:

- **`shared`**: Contains common logic, domain models, and generated Protobuf code.
    - Uses `prost` for Protobuf code generation (`build.rs`).
    - Defines the `MessageWrapper` enum for UDP wire format.
- **`backend`**: The server-side application.
    - **Framework**: `axum` for HTTP and WebSocket APIs.
    - **Runtime**: `tokio` for async execution and UDP handling.
    - **Responsibilities**:
        - Listens for UDP packets from the Realtime component.
        - Maintains shared state (`AppState`).
        - Broadcasts updates to the frontend via WebSockets.
        - Receives commands from the frontend and sends them via UDP.
- **`frontend`**: The client-side web application.
    - **Framework**: `leptos` (WASM).
    - **Responsibilities**:
        - Displays real-time data (charts, gauges).
        - Sends user commands to the backend via WebSockets.

## Tech Stack
- **Language**: Rust (2021 edition)
- **Web Frameworks**: Axum (Backend), Leptos (Frontend)
- **Communication**: UDP, WebSockets
- **Serialization**: Protobuf (`prost`), JSON (`serde`)
- **Deployment**: Docker (Multi-stage build, `debian:bookworm-slim`)

## Coding Guidelines

### General Rust
- **Clippy**: Ensure code passes `cargo clippy` without warnings.
- **Formatting**: Use `cargo fmt` to maintain consistent style.
- **Error Handling**: Use `Result` and `Option` idiomatically. Create custom error types (e.g., `thiserror`) where appropriate.

### Backend (Axum/Tokio)
- **State Management**: Use `Arc<RwLock<T>>` or `DashMap` for shared state accessible across handlers and tasks.
- **Async**: Spawn dedicated Tokio tasks for long-running services like the UDP listener.
- **Handlers**: Keep handlers thin; move business logic to service modules or domain methods.

### Frontend (Leptos)
- **Signals**: Use signals for reactive state management.
- **Components**: Break UI into small, reusable components.
- **WASM**: Remember that this code runs in the browser. Use `web-sys` for DOM interaction if necessary.

### Shared
- **Protobuf**: The `operSystem_api_realtime.proto` file is the source of truth for the UDP protocol. Re-run build if this file changes.

## Common Tasks

### Running the Project
*Note: Specific commands depend on the final build tool setup (e.g., `cargo-leptos`, `trunk`, or manual `wasm-pack`).*

- **Run Backend**:
  ```bash
  cargo run -p backend
  ```

- **Run Frontend** (assuming Trunk):
  ```bash
  trunk serve --open
  ```

### Testing
- **Unit Tests**:
  ```bash
  cargo test
  ```
- **Shared Crate Tests**:
  ```bash
  cargo test -p shared
  ```

## File Structure Highlights
- `plan.md`: Comprehensive implementation plan.
- `shared/build.rs`: Protobuf compilation script.
- `backend/src/udp.rs`: UDP communication logic.
- `frontend/src/components/`: UI components.

## Documentation Workflow
- **Project History**: After each prompt or significant task, update `project_history.md` with a concise summary of changes and decisions. This file serves as a source for the user's Google Doc project summary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AsafTzarfati)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AsafTzarfati)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
