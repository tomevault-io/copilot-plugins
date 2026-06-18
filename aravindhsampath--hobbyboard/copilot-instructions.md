## hobbyboard

> A selfhosted private Pinterest alternative.

# Project: Hobbyboard
A selfhosted private Pinterest alternative. 


# Contributing
- Create a branch for every new logical feature/change. Remind the user and Merge to main branch only upon approval.
- Keep tests up to date. Add new tests whenever new features or functions are added.
- Run `make all` to check, link, format and test after any changes.
- Use context7 to get up to date libraries and docs. Use latest versions of tools and libraries wherever possible.
- When the prompt says "plan" or "review", do not make changes without explicit approval.

# Technical details

## Architecture
HobbyBoard acts as a bridge between your local media files and a rich, searchable interface. It is built as a single Rust binary that orchestrates media processing, AI analysis, and web serving.

- **Backend:** Rust (Axum, Tokio, SQLx)
- **Frontend:** Vanilla JavaScript (ES6 modules, no bundler required)
- **Databases:**
  - **SQLite:** Stores file metadata, user tags, boards, and Full-Text Search (FTS5) indices.
  - **USearch:** Embedded vector index for semantic search and similarity matching (stored as a local file).
- **AI Stack:**
  - **Vision:** Connects to Ollama (local), OpenAI, or Gemini to analyze images/videos and generate captions, tags, and OCR text.
  - **Embeddings:** Uses `fastembed-rs` to generate vector embeddings locally within the Rust process.

## Core Workflows

### 1. Initialization (`init`)
Sets up the environment for a new library:
- Creates the SQLite database and schema.
- Initializes the USearch index file for vector storage.

### 2. Build / Indexing (`build`)
The heart of the ingestion process:
- **Scans:** Walks the `raw_images` directory to find new media.
- **Process:** Generates optimized thumbnails and video previews into `dist/`.
- **Analyze:** Sends media to the configured Vision AI provider to generate a detailed description and tags.
- **Embed:** Converts the AI-generated description into a vector using `fastembed-rs` and stores it in USearch.
- **Index:** Saves all metadata to SQLite, updating the FTS5 index for fast text search.
- **Refresh:** The `--refresh` flag allows wiping generated metadata (AI results) while preserving user data (manual tags, boards).

### 3. Serving (`serve`)
HOSTS the web interface and API:
- Serves static frontend files.
- Provides endpoints for search, media retrieval, and tag/board management.
- **Search Strategy:** Hybrid approach combining:
  - **Semantic Search:** Vector similarity via USearch (finds "conceptually" similar images).
  - **Text Search:** FTS5 query against titles, tags, and OCR text.
  - **Filtering:** Post-filtering by tags or boards (future enhancement: USearch native closures).

## Data Model (SQLite)
- **`items`**: core registry of files (paths, hashes, sizes).
- **`generated_metadata`**: AI outputs (captions, auto-tags). Volatile (can be re-generated).
- **`user_tags`**: Manual user tags. Persistent.
- **`boards` & `board_items`**: User collections. Persistent.
- **`media_search_fts`**: Virtual table for high-performance text queries.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

---
> Source: [aravindhsampath/hobbyboard](https://github.com/aravindhsampath/hobbyboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
