## absolutelyright

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A fun web application that tracks how many times Claude Code tells the user they are "absolutely right". It consists of:
- **Backend**: Rust/Axum server with in-memory storage
- **Frontend**: Static HTML/JS/CSS served by the backend

## Development Commands

### Build and Run
```bash
# Build the Rust backend
cargo build

# Run the server (serves on port 3003)
cargo run

# Build for release
cargo build --release
```

### Code Quality
```bash
# Format Rust code
cargo fmt

# Run clippy linter
cargo clippy

# Check for compilation errors
cargo check
```

## Architecture

### Backend (`src/main.rs`)
- **Framework**: Axum web framework with Tokio async runtime
- **Storage**: SQLite database (`counts.db`) using tokio-rusqlite for async operations
- **API Endpoints**:
  - `GET /api/today` - Returns today's count
  - `GET /api/history` - Returns all historical data
  - `POST /api/set` - Updates count for a specific day
- **Static Files**: Serves frontend from `frontend/` directory via Tower-HTTP

### Frontend (`frontend/`)
- **index.html**: Main page displaying current count and chart
- **frontend.js**: Fetches data from API and renders historical chart using SVG
- **style.css**: Styling for the application

## Key Implementation Details

- The server binds to `0.0.0.0:3003` for network accessibility
- Uses `chrono` for date handling with `%Y-%m-%d` format
- Frontend auto-refreshes count every 5 seconds
- Chart visualization is built with raw SVG (no external chart library)
- Data persistence via SQLite database with automatic table creation on startup
- Database schema: `day_counts` table with `day` (TEXT PRIMARY KEY) and `count` (INTEGER)

## Notes

- Frontend is served directly by the Axum server (no separate build process)
- No tests are currently implemented
- Database file `counts.db` is created automatically on first run
- Stop telling me I'm absolutely right.

---
> Source: [yoavf/absolutelyright](https://github.com/yoavf/absolutelyright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
