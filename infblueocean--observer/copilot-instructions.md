## observer

> **Observer** is an ambient news aggregation Terminal User Interface (TUI) built with **Go 1.24**. It aggregates content from over 200 sources (RSS/API) via the local `Clarion` library, embeds them using Jina AI, stores them in a hybrid storage system (SQLite + Milvus), and presents them in a real-time Bubble Tea interface.

# Observer Project (Gemini Context)

## Project Overview

**Observer** is an ambient news aggregation Terminal User Interface (TUI) built with **Go 1.24**. It aggregates content from over 200 sources (RSS/API) via the local `Clarion` library, embeds them using Jina AI, stores them in a hybrid storage system (SQLite + Milvus), and presents them in a real-time Bubble Tea interface.

**Core Philosophy:** Radical transparency. No hidden algorithms; all filters and source health states are visible.

## Architecture

The system is designed as a coupled, event-driven pipeline:

*   **`Clarion` (Source Engine)**:
    *   Independent worker goroutines per source (`internal/clarion/workers.go`).
    *   Adaptive scheduling: Tightens interval on new items, backs off on empty runs or errors.
    *   **User-Level Sources**: Supports `feeds.yaml` with shorthand prefixes (`reddit/`, `twitter/`, `telegram/`, `rsshub/`) and canonical URL resolution (`internal/feeds/`).
    *   Reliability: Implements browser-like User-Agents to bypass 403 blocks.
*   **`Intake` (Streaming Pipeline)**:
    *   `RunStream` processes events from Clarion with a **250ms Nagling buffer** to ensure efficient 32-item batching.
    *   **Stable Identity**: `EntryID` is source-agnostic (deduplicates across different source names) and uses case-safe URL canonicalization (stripping UTMs).
    *   **Safety**: HTML sanitization includes a 30k character "token bomb" limit.
*   **`Store` (Metadata & Persistence)**:
    *   Pure-Go SQLite (`modernc.org/sqlite`).
    *   **High Performance**: `InsertEntryBatch` commits all feed items in a single atomic transaction, reducing disk syncs by 50x.
    *   `source_state` table tracks per-source health, latency, and scheduling.
*   **`VecDB` (Vector Search)**:
    *   `Milvus` backend for cosine similarity search and cross-source deduplication.
    *   Deduplication (`DupCount`) happens at ingestion time with a ~0.95 similarity threshold.
*   **`UI` (Bubble Tea TUI)**:
    *   Real-time "Firefly" behavior: New entries pop in as they are ingested.
    *   **Performance**: Calculations (Bayesian MPM, history trimming) moved to `Update` loop; `View()` is a pure rendering function.
    *   Telemetry: `p/q/obs/est` format (Last Burst / Age / Trailing Minute / Bayesian Session Rate).

## Key Workflows

1. **Ingestion Stream**: Clarion worker fetches -> Intake inserts to SQLite (Batch) -> Emits UI signal -> Nagling Buffer -> Sync worker embeds (Jina) and pushes to Milvus (Batch 32).
2.  **Adaptive Scheduling**:
    *   `Hot`: 20s - 30s interval when new items are found.
    *   `Quiet`: Exponential backoff up to 15m when empty.
    *   `Error`: Exponential backoff up to 30m with jitter.
3.  **Search Pipeline**:
    *   **Fast**: Cosine similarity search on Milvus.
    *   **Precise**: FTS5 on SQLite for keyword matching.
4.  **Retention**: Background cleanup deletes entries > 7 days old (if not saved) from both SQLite and Milvus.

## Build and Run

### Prerequisites
*   Go 1.24+
*   `JINA_API_KEY` (Required for embedding).
*   Milvus instance (endpoint and API key).

### Commands
*   **Build TUI**: `go build -o observer ./cmd/observer`
*   **Build Debug CLI**: `go build -o obs ./cmd/obs`
*   **Run Tests**: `go test ./...`
*   **Check Stats**: `./obs stats --db`

## Development Conventions

*   **Go Style**: Standard formatting (`gofmt`), Go 1.24 features (e.g., `range` over functions).
*   **Event-Driven UI**: Never import `bubbletea` in `internal/`. Use `UISignalSink` (callbacks) to bridge intake to the TUI.
*   **Persistence**: Append-only design for entries. Use `sync_status` (`pending` -> `embedded` -> `synced`) to track pipeline progress.
*   **Testing**: Co-located `_test.go` files. Use table-driven tests for logic (especially scheduler and backoff).
*   **Commits**: Format: `area: summary` (e.g., `intake: add priority sync worker`).

## Directory Structure

*   `cmd/observer/`: TUI entry point and UI bridge.
*   `cmd/obs/`: Maintenance CLI (stats, backfill, search debug).
*   `internal/clarion/`: Source worker runtime and Clarion integration.
*   `internal/feeds/`: User-level source loading and shorthand resolution.
*   `internal/intake/`: Streaming ingestion, Nagling buffer, and sync coordination.
*   `internal/store/`: SQLite schema, batch persistence, and metadata CRUD.
*   `internal/milvus/`: Milvus vector provider implementation.
*   `internal/ui/`: Bubble Tea components and views.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/infblueocean) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
