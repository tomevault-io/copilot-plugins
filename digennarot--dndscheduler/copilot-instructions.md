## dndscheduler

> This project is a sophisticated web application for scheduling Dungeons & Dragons sessions. It features a Rust backend (Axum) and a vanilla JavaScript frontend with rich visual effects (Anime.js, p5.js). The application focuses on providing a "mystical" user experience while handling complex scheduling logic.

# D&D Session Scheduler - Developer Context

## Project Overview
This project is a sophisticated web application for scheduling Dungeons & Dragons sessions. It features a Rust backend (Axum) and a vanilla JavaScript frontend with rich visual effects (Anime.js, p5.js). The application focuses on providing a "mystical" user experience while handling complex scheduling logic.

### Tech Stack
*   **Backend:** Rust
    *   **Framework:** `axum` (0.7)
    *   **Runtime:** `tokio`
    *   **Database:** `sqlite` via `sqlx` (0.8.2)
    *   **Serialization:** `serde`, `serde_json`
    *   **Logging:** `tracing`
    *   **Auth:** Custom JWT/Session-based + Authelia integration
*   **Frontend:**
    *   **Structure:** HTML5 (served from `static/`)
    *   **Styling:** Tailwind CSS (custom config inferred), vanilla CSS (`static/css/`)
    *   **Logic:** Vanilla ES6+ JavaScript (`static/js/`)
    *   **Libraries:** Anime.js, p5.js, ECharts.js
*   **Infrastructure:** Docker (implied by docs), Caddy (reverse proxy/HTTPS), Authelia (SSO)

## Architecture

### Backend (`src/`)
The backend is a monolithic Axum service that handles API requests and serves static files.
*   **Entry Point:** `src/main.rs` initializes the database, configures logging, sets up middleware (CORS, Rate Limiting), and starts the Axum server on port 3000.
*   **Database:** `src/db.rs` manages the SQLite connection pool and handles schema initialization (inline migrations).
*   **Authentication:** `src/auth.rs` implements user registration, login (bcrypt), session management, and password policies (zxcvbn).
*   **Handlers:**
    *   `src/handlers.rs`: General API endpoints (polls, participation).
    *   `src/activity_handlers.rs`: Activity feed and reminders.
    *   `src/admin_stats.rs`: Admin dashboard statistics.
*   **Security:** `src/security.rs` likely handles headers and specific security configurations. `tower_governor` is used for rate limiting.

### Frontend (`static/`)
The frontend is a Multi-Page Application (MPA) where each HTML file corresponds to a specific view.
*   **`index.html`**: Dashboard.
*   **`create-poll.html`**: Wizard for creating new schedules.
*   **`participate.html`**: Interface for users to vote on times.
*   **`manage.html`**: Admin/Organizer view for a specific poll.
*   **`js/app.js`**: Main controller and global state.
*   **`js/particles.js`**: Visual effects.

## Development Workflow

### Prerequisites
*   Rust (latest stable)
*   SQLite
*   Optional: `cargo-watch` for hot reloading, `cargo-tarpaulin` for coverage.

### Building and Running
1.  **Setup Environment:**
    Ensure a `.env` file exists (check `.env.example`).
    ```bash
    cp .env.example .env
    ```

2.  **Run Application:**
    ```bash
    cargo run
    ```
    The server will start at `http://localhost:3000`.

3.  **Run Tests:**
    Use the provided script for convenience:
    ```bash
    ./scripts/run_tests.sh          # Run all tests
    ./scripts/run_tests.sh unit     # Run unit tests only
    ./scripts/run_tests.sh auth     # Run auth integration tests
    ```
    Or standard cargo:
    ```bash
    cargo test
    ```

### Database
The project uses SQLite. The database file `dnd_scheduler.db` is created automatically in the root directory if it doesn't exist. Schema updates are currently handled via `CREATE TABLE IF NOT EXISTS` queries in `src/db.rs`.

## Key Conventions
*   **Error Handling:** Custom `ErrorResponse` struct for JSON errors.
*   **Rate Limiting:** Distinct limits for Auth (strict) vs General API (lenient).
*   **Logging:** Structured logging via `tracing`. Use `RUST_LOG` to control verbosity.
*   **Security:** Strong password enforcement (12+ chars, entropy check), extensive audit logging (`src/audit.rs`).

## Directory Structure
*   `src/`: Rust source code.
*   `static/`: Frontend assets (HTML, CSS, JS, Images).
*   `docs/`: Extensive documentation on features, security, and design.
*   `scripts/`: Utility scripts for testing and maintenance.
*   `tests/`: Integration tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digennarot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/digennarot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
