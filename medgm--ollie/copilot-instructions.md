## ollie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Ollie** is a local-first desktop AI assistant for Linux built with Tauri v2 (Rust backend) + React 19 (TypeScript frontend). It supports local LLMs via Ollama and cloud providers (OpenAI, Anthropic, Google Gemini, GroqCloud).

## Commands

### Development

```bash
# Start full dev environment (Tauri launches Vite automatically)
cargo tauri dev

# Frontend only (from app/)
cd app && npm run dev

# Build for production
cargo tauri build
```

### Frontend (from `app/`)

```bash
npm run lint       # ESLint
npm run build      # tsc + vite build
```

### Backend (Rust)

```bash
cargo check        # Type-check without building
cargo clippy       # Lints
cargo test         # Run tests
```

## Architecture

### Monorepo Structure

- `app/` — React 19 + TypeScript frontend (Vite)
- `src-tauri/` — Rust backend (Tauri v2)

### Communication (IPC)

Frontend calls backend via `invoke('command_name', args)`. Real-time streaming uses Tauri events:
- `chat-chunk` — streaming token
- `chat-error` — stream error
- `chat-complete` — stream finished

### Backend Layout (`src-tauri/src/`)

- `commands/` — Tauri command handlers: `chat.rs`, `models.rs`, `db.rs`, `settings.rs`, `monitoring.rs`, `sys.rs`, `mcp.rs`
- `providers/` — Multi-provider abstraction: `traits.rs` defines `LLMProvider` trait; `orchestrator.rs` routes to `ollama.rs`, `openai.rs`, `anthropic.rs`, `google.rs`
- `db/` — SQLite connection pool + schema (WAL mode; tables: `chats`, `messages`)
- `mcp/` — Model Context Protocol: `protocol.rs` (message types), `transport.rs` (stdio/SSE)

### Frontend Layout (`app/src/`)

- `store/` — Zustand stores: `chatStore.ts`, `settingsStore.ts`, `modelsStore.ts`, `monitoringStore.ts`, `mcpStore.ts`, `uiStore.ts`, `setupStore.ts`
- `components/` — React UI components
- `routes/` — Page components: `chat.tsx`, `models.tsx`, `settings.tsx`
- `lib/` — Utilities (markdown, PDF, keyboard shortcuts, hooks)
- `types/` — TypeScript interfaces

### Key Patterns

- **View routing**: No router library — `uiStore.view` controls which page renders (`chat | models | settings | monitoring`)
- **Provider config**: `ProviderConfig` (id, name, type, credentials, base_url) stored in settings; `ChatOrchestrator` selects provider at runtime
- **Database path**: `~/.config/ollie/app.db`
- **Streaming cancellation**: `Arc<AtomicBool>` cancel tokens on the Rust side; frontend calls `chat_cancel()`
- **Backend results**: Rust commands return `Result<T, String>`; errors propagate as strings to frontend

### Adding a New Provider

1. Implement `LLMProvider` trait in `src-tauri/src/providers/`
2. Add variant to `ProviderType` enum in `providers/mod.rs`
3. Register in `orchestrator.rs`
4. Add frontend UI in settings store/components

---
> Source: [MedGm/Ollie](https://github.com/MedGm/Ollie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
