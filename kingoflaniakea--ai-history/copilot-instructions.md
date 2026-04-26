## ai-history

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI History is a local, offline AI conversation archival platform. It captures conversations from ChatGPT, Gemini, and AI Studio via a browser extension, sends them to a Tauri desktop app over a local HTTP bridge (127.0.0.1:48765), and stores them in SQLite with FTS5 full-text search.

## Monorepo Structure

- `apps/desktop` — Tauri desktop app (React frontend in `src/`, Rust backend in `src-tauri/`)
- `apps/extension` — Chrome/Edge browser extension (WXT framework, Manifest V3)
- `packages/core-types` — Shared TypeScript types (SourcePlatform, Conversation, Message, etc.)
- `packages/parsers` — Import parsers for ChatGPT, Gemini, AI Studio, HTML, Markdown formats
- `packages/ui` — Shared React components
- `packages/test-fixtures` — Sample data for parser tests

## Commands

```bash
pnpm install                # Install all dependencies
pnpm dev:desktop            # Vite dev server only (React frontend, port 1420)
pnpm dev:app                # Full Tauri dev mode (frontend + Rust backend)
pnpm dev:extension          # Build extension in watch mode
pnpm test:parsers           # Run parser unit tests (Vitest)
pnpm build                  # Build all packages
pnpm lint                   # Lint all packages
pnpm check:no-generated-js  # CI guard: ensures no .js files in src/ directories
```

Run a single parser test file:
```bash
pnpm --filter @ai-history/parsers exec vitest run tests/parsers/parser-registry.test.ts
```

## Tech Stack

- **Package manager**: pnpm 9.12.0 with workspaces
- **Frontend**: React 18, Vite 6, TypeScript 5.7 (strict mode), TanStack React Query, Zustand
- **Desktop backend**: Rust (Tauri 2), SQLite with FTS5, Axum HTTP server, Tokio async runtime
- **Extension**: WXT 0.20.6, Chrome Manifest V3, content scripts per platform
- **Testing**: Vitest 2.1.8, test fixtures in `packages/test-fixtures/`
- **CI**: GitHub Actions — pnpm install, JS guard check, parser tests (Node 20)

## Architecture

**Data flow**: Extension content scripts extract conversations from web pages → background service worker sends to desktop via HTTP bridge → Tauri Rust backend validates session token, parses, stores in SQLite → React frontend queries and displays.

**Key files**:
- `apps/extension/entrypoints/lib/extractor.ts` — Core extraction logic (~2200 lines)
- `apps/desktop/src-tauri/src/db/mod.rs` — SQLite database layer (~2300 lines)
- `apps/desktop/src-tauri/src/commands/mod.rs` — Tauri IPC command handlers
- `apps/desktop/src-tauri/src/http/mod.rs` — HTTP bridge server (Axum, CORS, session tokens)
- `packages/parsers/src/parser-registry.ts` — Parser selection by file format
- `packages/core-types/src/types.ts` — Shared type definitions

**Bridge protocol**: The extension communicates with the desktop app via HTTP on 127.0.0.1:48765 using session tokens with 10-minute expiry.

## Important Conventions

- All source code is TypeScript (strict mode) — CI enforces no `.js` files in `src/` directories
- Project documentation and UI are in Chinese
- Database lives at `~/.config/com.aihistory.desktop/ai-history.sqlite`
- ESLint/Prettier are not yet configured (lint scripts are placeholders)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KingofLaniakea) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
