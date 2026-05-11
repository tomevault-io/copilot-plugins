## gemini-sql

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Development (runs both Vite frontend and Tauri backend)
npm run tauri dev

# Build frontend only
npm run build

# Build production app (creates distributable)
npm run tauri build

# Build Rust backend only
cargo build --manifest-path src-tauri/Cargo.toml
```

## Architecture Overview

GeminiSQL is a cross-platform SQL editor built with **Tauri 2.0** (Rust backend) and **React 19** (TypeScript frontend).

### Frontend (`src/`)

**State Management** - Zustand stores in `src/stores/`:
- `connectionStore` - Database connections (active, list, connect/disconnect)
- `queryStore` - Query tabs, SQL content, execution
- `schemaStore` - Tables and columns for connected database
- `settingsStore` - Theme, API keys, AI model selection
- `aiStore` - AI assistant state, query generation
- `historyStore` - Query execution history

**Key Components** in `src/components/`:
- `editor/` - CodeMirror SQL editor with autocomplete
- `ai/` - AI Assistant drawer for query generation
- `results/` - Virtualized results table
- `sidebar/` - Tables list, saved queries, history tabs
- `connection/` - Connection dialog with MySQL support

### Backend (`src-tauri/src/`)

**Tauri Commands** in `commands/`:
- `connection.rs` - Connect, disconnect, test database connections
- `query.rs` - Execute SQL, manage saved queries and history
- `schema.rs` - Fetch tables and column metadata
- `ai.rs` - Google Gemini API integration for SQL generation

**Storage** in `storage/`:
- `database.rs` - SQLx connection pools for MySQL/PostgreSQL
- `local.rs` - SQLite for persisting connections, saved queries, history

**Data Flow**: Frontend calls Tauri commands via `@tauri-apps/api` invoke → Rust handlers in `commands/` → Database operations via `storage/`

## Key Technical Details

- **SQL Autocomplete**: Uses CodeMirror with custom completions from `schemaStore` tables/columns
- **AI Integration**: Google Gemini API with model selection (gemini-2.5-pro, gemini-2.5-flash, gemini-3-pro-preview)
- **MySQL Types**: DECIMAL handled via `rust_decimal`, TIMESTAMP via `chrono::DateTime<Utc>`
- **Results Table**: Virtualized with `@tanstack/react-virtual` for large result sets
- **Theme Colors**: Primary green `#3fb950`, stored in Tailwind config

## Current Limitations

- PostgreSQL support is disabled (Coming Soon badge in UI)
- Only MySQL connections are functional

---
> Source: [andrisasuke/gemini-sql](https://github.com/andrisasuke/gemini-sql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
