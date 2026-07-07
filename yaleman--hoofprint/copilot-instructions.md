## hoofprint

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is a multi-user web application written in Rust that is designed to store barcodes and QR codes for sites and shops. It allows users to create codes (barcodes/QR codes) that can be scanned at various sites to prove membership or access rights.

## Project Setup

This is a Rust project using:

- **axum-server** for the web server (with tokio-rustls for TLS)
- **SeaORM** with SQLite for database management
- **tokio** as the async runtime
- **tracing** for logging

The project uses Rust Edition 2024.

## Development Commands

### Building and Running

- `cargo build --quiet` - Build the project
- `cargo run --quiet` - Run the application
- `cargo run --quiet -- --debug` - Run with debug logging enabled
- `cargo run --quiet -- --database-file <path>` - Specify a custom database file path

### Testing

- `cargo test` - Run all tests
- `cargo test <test_name>` - Run a specific test
- `cargo test --lib` - Run only library tests
- `cargo test --test <integration_test>` - Run a specific integration test

### Code Quality

- `cargo clippy` - Run linter
- `cargo fmt` - Format code
- `cargo check` - Type check without building

## Architecture Overview

### Database Layer (`src/db/`)

The database layer is built on SeaORM with SQLite:

- **Connection Management** (`src/db/mod.rs`):
  - `connect()` establishes database connection and runs migrations
  - `test_connect()` provides in-memory database for tests
  - `get_connect_string()` handles both file-based and in-memory databases
  - Database file path is configurable via CLI arg `--database-file` or env var `HOOFPRINT_DB_FILE`

- **Entities** (`src/db/entities/`):
  - `code.rs` - Barcode/QR code entities
  - `site.rs` - Site/shop entities
  - Entity definitions should be kept in sync with migration schemas

- **Migrations** (`src/db/migrations/`):
  - Uses SeaORM migration framework
  - Each migration is a separate module (e.g., `m20251220_01.rs`)
  - Migrations run automatically on connection in a transaction
  - Current schema includes:
    - **Site** table: id (UUID), name, url, created_at
    - **User** table: id (UUID), display_name, email, groups (JSON), password
    - **Code** table: id (UUID), user_id (FK), type, value, site_id (FK), created_at, last_updated, name (optional)

### Configuration (`src/config.rs`)

- `Configuration` struct holds runtime config
- `SendableConfig` = `Arc<RwLock<Configuration>>` for thread-safe sharing
- Currently only stores `database_file` path

### CLI (`src/cli.rs`)

Uses clap for command-line argument parsing:

- `--debug` flag for debug logging
- `--database-file` for specifying database location (env: `HOOFPRINT_DB_FILE`, default: `./hoofprint.sqlite`)

### Prelude (`src/prelude.rs`)

Common imports available throughout the codebase:

- `Arc`, `RwLock` from tokio
- Configuration types
- Database `connect()` function

### Logging (`src/logging.rs`)

Initializes tracing/logging with support for debug mode.

## Database Workflow

When adding new database entities:

1. Create migration file in `src/db/migrations/` following naming convention `m<YYYYMMDD>_<NN>.rs`
2. Add migration to `Migrator::migrations()` in `src/db/migrations/mod.rs`
3. Define entity in `src/db/entities/`
4. Add entity module to `src/db/entities/mod.rs`
5. Migrations run automatically on application start via `connect()`

## Testing Strategy

- Use `test_connect()` for database tests - provides in-memory SQLite instance
- Tests are co-located with code using `#[cfg(test)]`
- Migration tests verify migrations can be applied successfully
- Always test database operations with the in-memory database before using real database

---
> Source: [yaleman/hoofprint](https://github.com/yaleman/hoofprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
