## spruceup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (requires Python 3.14)
uv sync

# Run the app (must be run from the directory containing spruceup_pipeline.py)
uv run spruceup start

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_sync_engine.py

# Run a specific test
uv run pytest tests/test_sync_engine.py::test_reconcile_new_file
```

Required env vars for the pipeline: `PG_CONNSTR`, and an embedder API key (e.g. `OPENAI_API_KEY`). Copy credentials into `.env`; `spruceup_pipeline.py` calls `dotenv.load_dotenv()` at import time.

## Architecture

SpruceUp is a document ingestion daemon. It watches source connectors for file changes, transforms documents into chunks, embeds them, and keeps a target vector store in sync.

### Pipeline file (`spruceup_pipeline.py`)

The user-authored entry point. The CLI (`spruceup start`) imports it dynamically from the CWD. It must define a `config` variable returned by `define_config()`:

```python
config = define_config(
    sources=[LocalFilesSource(watched_dir="example/data_corpus")],
    target=PgVectorTarget(connstr=..., table="data_chunks", schema=LectureChunk, vector_column="chunk_embedding"),
    embedder=OpenAIEmbedder(api_key=..., model="text-embedding-3-small"),
    transform=build_lecture_chunks,  # async fn(*, file_props: FileProps, embed) -> list[schema]
    cache_files=False,  # optional; True caches raw file content in the manifest (default False)
)
```

`define_config()` validates types eagerly at import time. `validate_pipeline()` in `cli.py` checks the contract exists before starting the event loop.

### Runtime flow (`app.py`)

On startup, `app.run(pipeline)` compares persisted fingerprints in the Manifest against the current config. Any mismatch triggers a **full reindex** (all files re-fetched, re-transformed, re-upserted) instead of incremental sync:
1. Transform function body changed (source hash)
2. Any `@memoize`-decorated function changed
3. Embedding model changed
4. Embedding dimensions changed
5. Target identity changed — `target.identity()`, a credential-free string (host/db/table or index/collection)
6. Schema changed — `hash_schema()` over field names+types and the designated `vector_column`

Signals 3–4 additionally **flush the embedding cache** (`embeddings_invalidated`). Signals 4–6 are **structural** and additionally **drop + recreate** the target table/index before reingest (`ensure_table_exists(recreate=True)`) — chosen over in-place migration because reingest must re-embed everything anyway.

On any mismatch, every file row is marked `needs_reindex` and the new fingerprints are persisted immediately (mark first, persist second — a crash in between just re-marks on the next start). A file stays `needs_reindex` until a sync **succeeds**; failures and restarts don't clear it. `SyncEngine.reconcile` pushes **all** chunks of a `needs_reindex` file instead of the diff (config changes don't alter chunk hashes, so the diff can't see them). `needs_reindex` files are re-enqueued at every startup and retried by the sync sweeper, so an interrupted reindex resumes where it left off.

Then it launches three concurrent asyncio tasks:

| Task | Role |
|------|------|
| `Monitor` | Runs all watchers; each watcher does a catch-up scan then enters a watch loop |
| `Coordinator` | Dequeues `SyncTask` objects and processes them (up to 32 concurrent) |
| `SyncSweeper` | Retries `failed` and `needs_reindex` files every 60 seconds |

### File change lifecycle

```
Source watcher → DebounceQueue → Coordinator
                                     ↓
                              source.fetch() → SpruceFile
                                     ↓
                              transform(file_props, embed) → list[UserChunk]
                                     ↓
                              SyncEngine.reconcile() → chunk diff → target.sync()
                                     ↓
                              Manifest.set_sync_state("synced")
```

`DebounceQueue` (wraps `asyncio.Queue`) evicts any already-queued task for the same `file_id` when a newer task arrives, preventing redundant processing. **Tradeoff:** to evict the superseded task it reaches into `asyncio.Queue` internals (`_queue`, `_unfinished_tasks`), so it's coupled to the CPython queue implementation and could break on a stdlib change.

### Manifest (`manifest.py`)

A local SQLite database (`spruceup_manifest.db`) that is the source of truth for:
- Registered data sources and their state (e.g. Google Drive page tokens)
- File rows: content hash, raw content (only when `cache_files=True`), sync state (`needs_reindex` / `in_flight` / `synced` / `failed`)
- Chunk rows: `(file_id, user_chunk_object_hash)` pairs for diffing
- Memoize cache: `(file_id, fn_hash, args_hash) → result`
- Embedding cache: `(file_id, chunk_text_hash) → embedding bytes`
- Config state: `embedding_model`, `embedding_dimensions`, `target_identity`, `schema_fingerprint`

Opened with `autocommit=True`; use `manifest.transaction()` only when multiple writes must be atomic.

### Connector ABCs (`connectors/base.py`)

All connectors implement one of three ABCs:

- **`SourceConnector`** — `source_type`, `source_identifier`, `create_watcher()`, `fetch()`, `validate()`, `is_supported()`, `decode_content()`
- **`TargetConnector`** — `vector_column`, `identity()`, `ensure_table_exists(recreate=False)`, `sync(upserts, deletes)`, `aclose()`
- **`EmbedderConnector`** — `embed_batch(batch)`, `process_chunks(chunks)`, `aclose()`

Available implementations:

| Type | Implementations |
|------|----------------|
| Source | `LocalFilesSource`, `GoogleDriveSource` |
| Target | `PgVectorTarget`, `PineconeTarget`, `WeaviateTarget` |
| Embedder | `OpenAIEmbedder`, `CohereEmbedder`, `GeminiEmbedder`, `VoyageAIEmbedder` |

`LocalFilesSource` and `LocalFileWatcher` exist for local testing. Production reasoning should be framed in terms of the connector ABCs.

An embedder's `api_key` accepts a `str` or a `Callable[[], str]` (e.g. a secrets-manager fetch, resolved at client build). On a credential rejection the embedder raises `TokenExpiredError`; the base `embed_batch_retrying` then drops the cached client so the next retry rebuilds it with a re-resolved token. Static-string keys are left untouched (no point re-resolving). Auth-error detection is per-SDK (each `embed_batch` catches its provider's exception and normalizes to `TokenExpiredError`).

### EmbeddingBatcher (`connectors/embedders/embedding_batcher.py`)

Wraps any `EmbedderConnector`. Accumulates chunks from concurrent file transforms and flushes them as batched API calls (max 100ms wait or `max_batch_size` chunks, max 5 concurrent API calls). Also consults the Manifest embedding cache before calling the API — cache is scoped per `file_id` and keyed by `blake2b(chunk_text)`.

Each accumulated chunk gets its own `asyncio.Future`; a per-call `asyncio.gather` over those futures reassembles a caller's embeddings in order and waits for completion, so the batcher carries no per-file slot bookkeeping (a flushed batch may mix chunks from several callers).

### `@memoize` decorator (`memoize/decorator.py`)

Caches async subfunctions in the Manifest, scoped per file. The decorated function must be `async` — decorating a sync function raises `TypeError` at import. Results are invalidated when the function body changes. Valid **only** when called from within the `transform` function — it reads the `contextvars` set by `Coordinator.upsert_file()` via `transform_scope`.

```python
@memoize(return_type=str)
async def summarize(text: str) -> str: ...
```

Supported return types: `str`, `int`, `float`, `bool`, `list`, `dict`.

### PgVectorTarget schema mapping

`ensure_table_exists()` inspects the user dataclass with `typing.get_type_hints()` and maps Python types to Postgres types. The embedding column is named **explicitly** via the target's `vector_column=` (validated at construction to be a `list[float]` field) and becomes `vector(N)` (requires `pgvector` extension), where `N` is the embedder's `embedding_dimensions`. Any *other* `list[float]` field maps to a plain `DOUBLE PRECISION[]` array, not a vector. The `id` column is always `TEXT PRIMARY KEY`, set to `f"{file_id}:{chunk.user_chunk_object_hash.hex()}"` (keyed per file). Upserts use `ON CONFLICT (id) DO UPDATE` so re-embeds (e.g. after a model change) overwrite existing rows.

### Google Drive source

`GoogleDriveSource` takes a `watched_dir` (folder ID) and an `on_token_expired: Callable[[], str]` that returns a fresh OAuth access token. The `drive.readonly` scope covers all required API calls (list, download, export, changes). Startup validation rejects nested watched folders.

---
> Source: [SpruceUp-ai/SpruceUp](https://github.com/SpruceUp-ai/SpruceUp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
