## naive-elfiee

> **Elfiee** is a block-based editor for the `.elf` file format, designed with a focus on literate programming and event sourcing. It allows users to edit documents composed of various block types (markdown, code, diagrams) where every change is captured as an immutable event.

# Elfiee Project Context

## Project Overview

**Elfiee** is a block-based editor for the `.elf` file format, designed with a focus on literate programming and event sourcing. It allows users to edit documents composed of various block types (markdown, code, diagrams) where every change is captured as an immutable event.

The system is built on a **Capability-based Architecture**, meaning functionality is defined by "Capabilities" (like `markdown.write` or `code.execute`) that are dynamically granted to editors.

**Key Architectural Concepts:**
*   **Block-based Editing:** Content is structured in typed blocks.
*   **Event Sourcing:** All changes are stored as an append-only log of events in a SQLite database within the `.elf` (zip) file.
*   **Actor Model:** Each open file is managed by a dedicated Engine Actor in the backend to handle concurrency and state projection.
*   **CBAC (Capability-Based Access Control):** Granular permission system for editors.

## Tech Stack

### Frontend
*   **Framework:** React (v18)
*   **Language:** TypeScript
*   **Build Tool:** Vite
*   **State Management:** Zustand
*   **Styling:** Tailwind CSS, Shadcn UI, Radix UI
*   **Rendering:** Myst Parser (for markdown/literate programming)

### Backend
*   **Framework:** Tauri v2
*   **Language:** Rust
*   **Database:** SQLite (via `sqlx`)
*   **File Format:** ZIP archive (containing SQLite db and assets)
*   **Type Sharing:** `specta` / `tauri-specta` (generates TS types from Rust structs)

## Project Structure

*   `src/` - Frontend source code (React/TypeScript).
*   `src-tauri/` - Backend source code (Rust).
    *   `src/commands/` - Tauri commands callable from frontend.
    *   `src/models/` - Data models (Block, Event, Editor, etc.).
    *   `src/extensions/` - Built-in extensions and capability implementations.
*   `elfiee-ext-gen/` - A Rust-based CLI tool for scaffolding new extensions.
*   `docs/` - Comprehensive documentation on architecture, migration plans, and concepts.
*   `Makefile` - Automation for development tasks.

## Building and Running

The project uses `make` and `pnpm` for task management.

### Prerequisites
*   Node.js & pnpm
*   Rust (Cargo)

### Key Commands

*   **Start Development Server:**
    ```bash
    make dev
    # or
    pnpm tauri dev
    ```
    This starts the Tauri application with hot-reloading for the frontend.

*   **Run All Tests:**
    ```bash
    make test
    ```
    Runs both Rust backend tests (`cargo test`) and frontend tests (`vitest`).

*   **Format Code:**
    ```bash
    make fmt
    ```
    Formats Rust code (via `rustfmt`) and frontend code (via `prettier`).

*   **Clean Build Artifacts:**
    ```bash
    make clean
    ```

## Development Conventions

*   **Event Sourcing:** State is never mutated directly. Use capabilities to generate events, which are then applied to the state projector.
*   **Type Safety:** Use `tauri-specta` to share types between Rust and TypeScript. Run `cargo test` or `cargo check` to regenerate bindings if Rust types change.
*   **Extensions:** New block types should be implemented as extensions. Use `elfiee-ext-gen` to scaffold them.
*   **Formatting:** Always run `make fmt` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/H2OSLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
