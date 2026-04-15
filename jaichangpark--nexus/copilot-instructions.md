## nexus

> **Nexus** is a modern workflow builder application designed with a full-stack architecture. It features a reactive frontend for designing workflows and a robust Rust backend for execution and state management.

# Nexus Project Context

## Project Overview

**Nexus** is a modern workflow builder application designed with a full-stack architecture. It features a reactive frontend for designing workflows and a robust Rust backend for execution and state management.

### Key Technologies

*   **Frontend:** Svelte 5 (SvelteKit), TypeScript, Tailwind CSS v4, Vite.
*   **Backend:** Rust (Edition 2024), Axum Web Framework, Tokio Async Runtime.
*   **Infrastructure:** Docker Compose for orchestration.
*   **Package Manager:** Bun (for frontend).

## Architecture & Directory Structure

```text
/Users/jaichang/IdeaProjects/nexus/
├── frontend/           # SvelteKit Application
│   ├── src/
│   │   ├── lib/        # Shared components, utilities, and flow nodes
│   │   ├── routes/     # File-based routing (pages and layouts)
│   │   └── app.html    # HTML shell
│   ├── package.json    # Dependencies and scripts (managed via Bun)
│   └── vite.config.ts  # Build configuration
├── nexus-core/         # Rust Backend Application
│   ├── src/
│   │   └── main.rs     # Application entry point
│   └── Cargo.toml      # Rust dependencies and configuration
├── ref/                # Reference Implementations
│   ├── dify/           # Reference: Dify workflow engine
│   ├── n8n/            # Reference: n8n workflow automation
│   └── rust-sdk/       # Reference: Rust SDK patterns
├── docker-compose.yml  # Container orchestration for full-stack development
├── AGENTS.md           # AI Assistant guidelines and context
└── GEMINI.md           # This context file
```

## Getting Started

### Prerequisites

*   **Rust:** Latest stable toolchain (`rustup update`).
*   **Bun:** Latest version (Javascript runtime & package manager).
*   **Docker:** For running the full stack via containers.

### Running the Application

#### Option 1: Docker (Recommended for Full Stack)

Run the entire stack (Frontend + Backend) with hot-reloading enabled for volumes.

```bash
docker-compose up
# OR detached
docker-compose up -d
```

*   **Frontend:** http://localhost:5173
*   **Backend:** http://localhost:3001

#### Option 2: Local Development (Manual)

**Backend (Rust):**

```bash
cd nexus-core
cargo run
# Server starts at http://localhost:3001
```

**Frontend (SvelteKit):**

```bash
cd frontend
bun install
bun dev
# Dev server starts at http://localhost:5173
```

## Development Workflows & Commands

### Frontend (`frontend/`)

*   **Start Dev Server:** `bun dev`
*   **Type Checking:** `bun run check` (uses `svelte-check`)
*   **Watch Mode (Types):** `bun run check:watch`
*   **Linting:** `bun run lint` (Prettier check)
*   **Formatting:** `bun run format` (Prettier write)
*   **Build:** `bun run build`

### Backend (`nexus-core/`)

*   **Run:** `cargo run`
*   **Check:** `cargo check` (Fast compilation check)
*   **Format:** `cargo fmt`
*   **Lint:** `cargo clippy`
*   **Test:** `cargo test`

## Coding Conventions

### General
*   **File Naming:**
    *   Svelte/TS: `kebab-case` for files, `PascalCase` for components.
    *   Rust: `snake_case` for files and modules.
*   **Formatting:** Prettier for Frontend, `rustfmt` for Backend. Run formatters before committing.

### Frontend (Svelte 5)
*   **State:** Use Svelte 5 Runes (`$state`, `$derived`, `$effect`) for reactivity.
*   **Styles:** Tailwind CSS v4. Use utility classes. Component-specific styles go in `<style>` blocks.
*   **Imports:**
    *   `$lib/*` for internal library code.
    *   Prefer explicit type imports (`import type { ... }`).

#### Best practices for lib/ organization
As your app grows, lib/ can get messy if you throw everything in the root. Common practice is to add subfolders:
- components/ → all your Svelte components
- stores/ → global state (using Svelte stores)
- utils/ → helper JS functions
- assets/ → icons, images, static files you want to import in code (different from /static, which is for files served as-is)

```
src/lib/
├── components/
│   ├── Counter.svelte
│   ├── Button.svelte
│   └── Navbar.svelte
├── stores/
│   └── user.js
├── utils/
│   └── formatDate.js
└── assets/
└── favicon.svg

```


### Backend (Rust/Axum)
*   **Async:** Uses `tokio` runtime.
*   **Web Framework:** `axum` 0.7.
*   **Error Handling:** Use `Result` types and `axum::http::StatusCode` for API responses.
*   **Serialization:** `serde` with `serde_json` for API payloads.

## References & Resources

*   **SvelteKit:** https://kit.svelte.dev/docs
*   **Svelte 5:** https://svelte.dev/docs/svelte/runes
*   **Axum:** https://docs.rs/axum/latest/axum/
*   **Tailwind CSS:** https://tailwindcss.com/docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JAICHANGPARK) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
