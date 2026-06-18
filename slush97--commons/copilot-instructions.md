## commons

> Self-hosted, privacy-first personal app ecosystem. "Quiet Tools" — Japanese stationery aesthetic.

# Commons

Self-hosted, privacy-first personal app ecosystem. "Quiet Tools" — Japanese stationery aesthetic.

## What This Is

A unified personal data platform: notes, calendar, tasks, photos, email, messages, files — all sharing a knowledge graph with vector embeddings, queryable by a local LLM (Ollama). Built from scratch, not a wrapper around existing apps.

## Architecture

- **Shared Rust core crate** (`crates/core`) — models, storage trait, sync, embeddings, query engine, crypto
- **Server** (`crates/server`) — Axum HTTP server, runs on homeserver (ThinkPad x220, i5, 8GB RAM target)
- **Android client** — Kotlin + Rust via UniFFI (not Flutter)
- **Desktop client** — Tauri (Rust + web frontend, not Electron)
- **LLM layer** — Ollama on gaming PC (5070ti, 32GB RAM), optional tier on top of always-on services

### Two-Tier Deployment
- **Tier 1 (ThinkPad)** — Server + services, always on, low power. Everything works without Tier 2.
- **Tier 2 (Gaming PC)** — Embedding pipeline, vector store, Ollama. Intelligence layer, catches up when powered on.

## Tech Stack

- Language: Rust (core + server), Kotlin (Android), Web tech in Tauri (desktop)
- Storage: SQLite (clients, offline-first), Postgres + pgvector (server)
- API framework: Axum
- Sync: CRDT-based, offline-first, phone syncs to server via HTTPS/WebSocket
- Embeddings: fastembed crate
- LLM: Ollama (local inference only)

## Project Structure

```
commons/
├── CLAUDE.md
├── ARCHITECTURE.md          — Mermaid diagrams, entity model, system design
├── design/
│   ├── DESIGN_SPEC.md       — Full UI/UX specification
│   └── tokens.json          — Machine-readable design tokens
└── crates/
    ├── core/                — Shared library
    │   └── src/
    │       ├── models/      — Person, Note, Task, Event, Photo, Email, Message, File, Tag, Location
    │       │                  Edge, EntityKind, Relation (knowledge graph)
    │       └── store/       — Store trait (async, Postgres + SQLite impls)
    └── server/              — Axum binary
```

## Entity Model

Every entity has: `uuid` (v7), `created_at`, `modified_at`, and most have an `embedding: Option<Embedding>` for semantic search. Entities are linked via `Edge` (source, target, relation) forming the knowledge graph.

Entities: Person, Note, Task, Event, Photo, Email, Message, Conversation, File, Tag, Location

## Design Direction — "Quiet Tools"

- Dark mode primary: bg #2A2E28, clay accent #C4724A, forest #6B8F6B, cream text #D4CCBC
- Typography: Inter (headings), Source Sans 3 (body), JetBrains Mono (code)
- Icons: Lucide, outlined, 1.5px stroke
- No gradients, no shadows. Border-only depth cues.
- Labels lowercase. No exclamation marks. Empty states are understated.
- See `design/DESIGN_SPEC.md` for full component specs.

## Build Order

1. Core crate models + SQLite store (notes first)
2. Server API routes for notes CRUD
3. Sync protocol (CRDT-based)
4. Tasks, Calendar, Contacts
5. Embedding pipeline + knowledge graph
6. LLM query integration
7. Photos, Email, SMS ingestion
8. Tauri desktop + Android clients

## Conventions

- Workspace managed via root `Cargo.toml`, all deps in `[workspace.dependencies]`
- UUIDs are v7 (time-sortable)
- All timestamps are UTC `DateTime<Utc>`
- Store trait is async, `#[allow(async_fn_in_trait)]`
- Do NOT add `Co-Authored-By` trailers to commits
- Keep the ThinkPad (8GB RAM) as a deployment target — server must be lightweight

---
> Source: [Slush97/commons](https://github.com/Slush97/commons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
