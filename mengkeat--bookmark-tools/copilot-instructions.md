## bookmark-tools

> This file contains detailed guidance for coding agents working in this repository.

# CODING AGENTS

This file contains detailed guidance for coding agents working in this repository.

---

## Context Specification Document

*Generated: 2026-03-29 | Updated: 2026-04-04 — bookmark-check CLI, batch import (--file), logging module, html.parser, HTTP retry, integration tests*

---

### Quick navigation (table of contents)

- [Read this first (progressive disclosure)](#read-this-first-progressive-disclosure)
- [1. System Purpose](#1-system-purpose)
  - [1.1 Development Commands](#11-development-commands)
- [2. Entry Points & Execution Flow](#2-entry-points--execution-flow)
- [3. Core Modules & Responsibilities](#3-core-modules--responsibilities)
- [4. Data Models & Schemas](#4-data-models--schemas)
- [5. External Dependencies & I/O](#5-external-dependencies--io)
- [6. Known Pain Points / Technical Debt](#6-known-pain-points--technical-debt)

### Read this first (progressive disclosure)

If you only need to make a small, safe change, read these sections in order and stop early when you have enough context:

1. **Section 1.1** — Development commands (`uv run ...`)
2. **Section 2** — Entry points + high-level execution flow
3. **Section 3** — Module table (open only the files you are changing)
4. **Sections 4–6** — Data models, I/O, and technical debt (deep dive only when needed)

Summary behavior at a glance:

1. `summarize` CLI
2. classifier-provided summary from `call_llm(...)`
3. direct LLM fallback (`summarize_with_llm(...)`)
4. heuristic fallback (`infer_summary(...)`)

Common edit recipes:

- **Tune folder/type/tags decisions:** start in `bookmark_tools/classify.py`.
- **Tune summary behavior:** start in `bookmark_tools/summarize.py`.
- **Tune CLI flow / dry-run output:** start in `bookmark_tools/cli.py`.
- **Verify quickly without writing files:** `uv run bookmark <URL> --dry-run`.

Files not to Commit:

- PLAN.md or other planning related files

---

### 1. System Purpose

**One-sentence description:** A CLI tool that fetches a web page by URL and creates a structured bookmark note in an Obsidian vault.

Classification is LLM-first (with heuristic fallback), and summary generation is `summarize`-first with classifier/LLM/heuristic fallbacks.

**Primary technology stack:**
- Python 3.12 (stdlib only — no third-party packages required at runtime)
- [uv](https://docs.astral.sh/uv/) for project and dependency management
- `urllib.request` for HTTP fetching (page content + LLM API)
- `html.parser` for robust HTML metadata extraction (title, meta tags, lang attribute)
- `subprocess` + `shutil.which` for optional `summarize` CLI invocation
- `re` + `html` for HTML cleaning (no BeautifulSoup/lxml)
- `json` for LLM request/response serialization
- `dataclasses` + `typing.TypedDict` for data models
- `logging` for structured diagnostic output across all modules
- Optional: any OpenAI-compatible chat completions API (`gpt-4.1-mini` default model; OpenAI/OpenRouter compatible config)
- Optional: external `summarize` CLI (`summarize --help`) for primary summary generation

### 1.1 Development Commands

Use `uv run` to execute tools inside the project environment:

```bash
# Run tests
uv run pytest tests/

# Lint Python code
uv run ruff check bookmark_tools tests

# Auto-format Python code
uv run ruff format bookmark_tools tests

# Verify formatting without writing changes
uv run ruff format --check bookmark_tools tests
```

---

### 2. Entry Points & Execution Flow

#### Primary entry point — bookmark creation (via uv)
```
uv run bookmark <URL> [--dry-run] [--disallow-new-subfolder]
```
Uses the `bookmark` script entry point defined in `pyproject.toml`, which calls `bookmark_tools:main`.

#### Alternative entry point — bookmark creation
```
uv run python -m bookmark_tools <URL> [--dry-run] [--disallow-new-subfolder]
```
Invokes `bookmark_tools/__main__.py` → calls `cli.main()`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `<URL>` | Yes | — | Web page URL to fetch and classify |
| `--file`, `-f` | No | None | Read URLs from a file (one per line); use `-` for stdin |
| `--dry-run` | No | False | Print target path + rendered note and exit; **does not write files** |
| `--disallow-new-subfolder` | No | False | Restrict classifier output to existing folders only |
| `--verbose`, `-v` | No | False | Enable verbose (debug) logging output |
| `--quiet`, `-q` | No | False | Suppress all logging output except errors |

#### Bookmark search entry point
```
uv run bookmark-search <QUERY> [--folder <FOLDER>] [--tag <TAG>] [--limit <N>] [--rebuild]
uv run bookmark-search <QUERY> --semantic [--threshold <FLOAT>] [--tag <TAG>] [--limit <N>]
uv run bookmark-search <QUERY> --hybrid [--threshold <FLOAT>] [--tag <TAG>] [--limit <N>]
uv run bookmark-search <QUERY> --show-chunks
```
Uses the `bookmark-search` script entry point defined in `pyproject.toml`, which calls `bookmark_tools.search:main`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `<QUERY>` | Yes | — | Search query text |
| `--folder` | No | None | Restrict results to a folder and its subfolders (e.g., `ML-AI`) |
| `--tag` | No | None | Restrict results to bookmarks with the given tag |
| `--show-chunks` | No | False | Return multiple matching chunks from the same bookmark instead of one best note result |
| `--limit` | No | 10 | Maximum number of results to return (must be ≥ 1) |
| `--rebuild` | No | False | Force a full FTS5 index rebuild instead of incremental update |
| `--semantic` | No | False | Use embedding-based semantic search instead of keyword FTS5 search |
| `--hybrid` | No | False | Combine BM25 and semantic search via Reciprocal Rank Fusion |
| `--threshold` | No | 0.40 | Minimum similarity score for semantic/hybrid results |

#### Alternative search entry point
```
uv run python -m bookmark_tools.search <QUERY> [--folder <FOLDER>] [--limit <N>]
```

#### Bookmark health check entry point
```
uv run bookmark-check [--timeout <N>] [--verbose] [--quiet]
```
Uses the `bookmark-check` script entry point defined in `pyproject.toml`, which calls `bookmark_tools.check:main`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--timeout` | No | 15 | Timeout in seconds for each URL check |
| `--verbose`, `-v` | No | False | Enable verbose (debug) logging output |
| `--quiet`, `-q` | No | False | Suppress all logging output except errors |

#### Derived state rebuild entry point
```
uv run bookmark-rebuild [--catalog] [--no-embeddings] [--json] [--verbose] [--quiet]
```
Uses the `bookmark-rebuild` script entry point defined in `pyproject.toml`, which calls `bookmark_tools.rebuild:main`.

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `--catalog` | No | False | Rebuild the unified catalog (bookmarks, FTS, embeddings, stubs) |
| `--no-embeddings` | No | False | Skip embedding rebuild |
| `--json` | No | False | Print a JSON result object |
| `--verbose`, `-v` | No | False | Enable verbose (debug) logging output |
| `--quiet`, `-q` | No | False | Suppress all logging output except errors |

#### High-level execution order

```
Step 1: load_env()                      [paths.py]       — Load .env files into os.environ
Step 2: parse_args()                    [cli.py]         — Parse CLI args (url, --dry-run, --disallow-new-subfolder)
Step 3: build_note(url, allow_new)      [cli.py]         — Orchestrator; returns (target_path, note_text, folder_message)
  ├─ 3a: collect_existing_notes()       [vault_profile]  — Walk Bookmarks/**/*.md → build BookmarkProfile + URL index
  ├─ 3b: find_existing_url(url, profile)[classify.py]    — O(1) duplicate check via profile.url_index (fallback scan if no profile)
  ├─ 3c: extract_page_data(url)         [fetch.py]       — HTTP GET → parse title, description, language, content (first 8KB)
  ├─ 3d: rank_similar_notes(page, prof) [classify.py]    — Token-overlap similarity ranking (top 8)
  ├─ 3e: call_llm(...)                  [classify.py]    — POST to LLM API; returns JSON metadata or None
  │       ├─ on error: print fallback reason to stderr
  │       └─ fallback: heuristic_classification(...)      — Deterministic classification when LLM unavailable
  ├─ 3f: generate_summary(...)          [summarize.py]   — `summarize` CLI first → classifier summary → LLM summary → heuristic fallback
  ├─ 3g: validate_folder(folder, allow) [classify.py]    — Enforce vault folder constraints; may remap folder
  ├─ 3h: normalize_metadata(...)        [cli.py]         — Fill/correct all metadata fields with typed helpers
  └─ 3i: render_note(metadata, url)     [render.py]      — Produce final Markdown string with YAML frontmatter
Step 4: Write note to disk (or print if --dry-run; no write)
```

---

### 3. Core Modules & Responsibilities

Start with rows marked **Mutable**. Most implementation changes should be confined there.

| File Path | Responsibility | Key Functions/Classes | Mutability |
|-----------|---------------|----------------------|------------|
| `bookmark_tools/cli.py` | Orchestration: argument parsing, workflow coordination, summary handoff, file writing, metadata normalization, batch import (`--file`), logging configuration | `main()`, `build_note()`, `normalize_metadata()`, `_normalize_text()`, `_normalize_text_list()`, `_normalize_related_topics()`, `parse_args()`, `configure_logging()`, `_read_urls_from_file()`, `_process_single_url()` | **Mutable** (business logic) |
| `bookmark_tools/fetch.py` | HTTP page fetching and HTML content extraction with retry support, HTML parser-based metadata extraction | `fetch_text(url) → (final_url, html_text)`, `extract_page_data(url) → PageData`, `search_meta()`, `clean_html()`, `_MetadataParser`, `_parse_metadata()` | Read-Only (utility) |
| `bookmark_tools/classify.py` | Classification logic: LLM-based and heuristic fallback, folder validation, similarity scoring, env config, retry support | `call_llm()`, `heuristic_classification()`, `rank_similar_notes()`, `strong_similar_notes()`, `choose_folder_from_profile()`, `derive_tags()`, `derive_related_topics()`, `derive_parent_topic()`, `enrich_tags_from_similar()`, `validate_folder()`, `find_existing_url()`, `get_llm_config()`, `related_note_count()`, `SimilarNote` | **Mutable** (core business logic) |
| `bookmark_tools/summarize.py` | Summary generation pipeline with explicit fallbacks, retry support, logging | `generate_summary()`, `summarize_with_tool()`, `summarize_with_llm()` | **Mutable** (business logic) |
| `bookmark_tools/check.py` | Bookmark health checking: HEAD request URL validation, batch problem reporting | `check_url()`, `check_bookmarks()`, `parse_args()`, `main()` | **Mutable** (business logic) |
| `bookmark_tools/http_retry.py` | HTTP retry with exponential backoff and full jitter for transient failures | `urlopen_with_retry()` | Read-Only (utility) |
| `bookmark_tools/types.py` | Shared typed schemas for bookmark pipeline data | `PageData`, `BookmarkMetadata`, `NormalizedBookmarkMetadata` | Read-Only (types) |
| `bookmark_tools/render.py` | Markdown note rendering with YAML frontmatter | `render_note(metadata, url, profile) → str`, `infer_summary()`, `slugify_filename()`, `uniquify_path()`, `yaml_scalar()`, `yaml_list()` | Read-Only (utility) |
| `bookmark_tools/paths.py` | Configurable path resolution, timeout/size constants, `.env` loader | `get_bookmarks_dir()`, `get_search_index_path()`, `get_guide_path()`, `DEFAULT_TIMEOUT` (20s), `MAX_FETCH_BYTES` (1MB), `load_env()` | Read-Only (config) |
| `bookmark_tools/vault_profile.py` | Vault introspection: reads existing notes, builds profile with schema/folders/examples/topics, URL index | `collect_existing_notes() → BookmarkProfile`, `parse_frontmatter()`, `read_frontmatter()`, `tokenize()`, `choose_schema()`, `list_existing_folders()`, `NoteProfile`, `BookmarkProfile` | Read-Only (data collection) |
| `bookmark_tools/__init__.py` | Package re-export | `main` (re-exported from cli) | Read-Only |
| `bookmark_tools/__main__.py` | Package runner | Calls `cli.main()` | Read-Only |
| `bookmark_tools/search.py` | Search orchestration: CLI argument parsing, BM25/semantic/hybrid search, result formatting, logging | `main()`, `search_bookmarks()`, `search_bookmarks_semantic()`, `search_bookmarks_hybrid()`, `_reciprocal_rank_fusion()`, `parse_args()`, `configure_logging()` | **Mutable** (business logic) |
| `bookmark_tools/catalog.py` | Unified SQLite catalog: schema versioning, migrations, bookmarks metadata, fetch_log, chunks, edges (populated via `graph.py`), jobs stub, compatibility wrappers | `connect()`, `ensure_catalog_schema()`, `rebuild_catalog()`, `delete_from_catalog()`, `upsert_bookmark()`, `populate_bookmarks()`, `get_catalog_info()`, `CatalogResult`, `CatalogInfo` | Read-Only (utility) |
| `bookmark_tools/search_index.py` | SQLite FTS5 index management: schema creation, chunked BM25-weighted search, query sanitization | `rebuild_search_index()`, `update_search_index()`, `search_index()`, `SearchResult` | Read-Only (utility) |
| `bookmark_tools/search_documents.py` | Document collection: reads vault notes and preserves body text for chunking in `SearchDocument` records | `collect_search_documents()`, `SearchDocument` | Read-Only (utility) |
| `bookmark_tools/chunking.py` | Section-aware chunk generation for FTS, embeddings, and catalog chunk rows | `chunk_document()`, `chunk_documents()`, `SearchChunk` | Read-Only (utility) |
| `bookmark_tools/embeddings.py` | Embedding-based semantic chunk search: OpenAI embedding API, vector storage in SQLite, cosine similarity ranking, retry support | `embed_texts()`, `build_embedding_chunk_text()`, `refresh_embeddings()`, `semantic_search()`, `EmbeddingMatch` | Read-Only (utility) |
| `bookmark_tools/link.py` | Link management: URL validation, link extraction, and bookmark link operations | **Mutable** (business logic) |
| `bookmark_tools/reorg.py` | Bookmark reorganization: folder restructuring, batch moves, vault reorganization | **Mutable** (business logic) |
| `bookmark_tools/stats.py` | Bookmark statistics: analytics, metrics collection, vault statistics reporting | **Mutable** (business logic) |
| `bookmark_tools/tag_normalize.py` | Tag normalization: tag standardization, deduplication, and consistency enforcement | **Mutable** (business logic) |
| `bookmark_tools/update.py` | Bookmark updates: URL updates, metadata refresh, batch update operations | **Mutable** (business logic) |
| `bookmark_tools/graph.py` | Deterministic graph: edge extraction (in_folder/has_tag/from_domain/links_to_url/related_to), edge table rebuild/upsert/delete, backlinks, related suggestions, BFS traversal | `extract_edges()`, `rebuild_edges()`, `upsert_edges_for_note()`, `replace_edges_for_bookmark()`, `get_backlinks()`, `get_related()`, `traverse()`, `Edge`, `Backlink`, `RelatedBookmark` | **Mutable** (business logic) |
| `bookmark_tools/graph_cli.py` | `bookmark-backlinks` and `bookmark-graph` commands: resolve URL/path to bookmark id, query edges, text/JSON output | `backlinks_main()`, `graph_main()` | **Mutable** (business logic) |
| `bookmark_tools/hubs.py` | Topic/domain/tag hub page generation with protected generated blocks, orphan pruning, `bookmark-topics` CLI | `build_hub_pages()`, `main()`, `HubResult` | **Mutable** (business logic) |
| `bookmark_tools/config/` | Configuration files and settings | Read-Only (config) |
| `tests/test_bookmarks.py` | Unit tests (unittest) for helpers + summary pipeline behavior | `BookmarkHelpersTest` (14 tests) | Test file |
| `tests/test_bookmark_search.py` | Unit tests (unittest) for search: document collection, BM25 ranking, folder filtering | `BookmarkSearchTest` | Test file |
| `tests/test_embeddings.py` | Unit tests (unittest) for embeddings: vector helpers, semantic search, RRF, hybrid search | `EmbeddingHelpersTest`, `EmbeddingIndexTest`, `ReciprocalRankFusionTest`, `HybridSearchTest` | Test file |
| `tests/test_integration.py` | Integration tests for full `build_note()` pipeline with mocked network | `BookmarkIntegrationTest` | Test file |
| `tests/test_http_retry.py` | Unit tests for HTTP retry: exponential backoff, jitter, retryable codes, non-retryable errors | `HttpRetryTest` | Test file |
| `tests/test_fetch.py` | Unit tests for fetch module: metadata parsing, HTML cleaning, page data extraction | `FetchTest` | Test file |
| `tests/test_catalog.py` | Unit tests for catalog: schema, migrations, CRUD, rebuild, upgrade paths | `CatalogConnectionTest`, `CatalogSchemaTest`, `PopulateBookmarksTest`, `UpsertBookmarkTest`, `DeleteFromCatalogTest`, `RebuildCatalogTest`, `GetCatalogInfoTest`, `CatalogResultTest`, `CatalogSchemaUpgradeTest` | Test file |
| `tests/test_check.py` | Unit tests for bookmark-check: URL checking, problem detection, edge cases | `BookmarkCheckTest` | Test file |
| `tests/test_note_filter.py` | Unit tests for note filter: archive sidecar detection, bookmark path iteration | `IsArchiveSidecarTest`, `IterBookmarkNotePathsTest` | Test file |
| `tests/test_paths.py` | Unit tests for paths: fail-fast vault validation, directory resolution | `RequireBookmarksDirTest`, `GetBookmarksDirTest` | Test file |
| `tests/test_url_normalize.py` | Unit tests for URL normalization: scheme/host, default ports, path, identity | `NormalizeUrlTest` | Test file |
| `tests/test_graph.py` | Unit tests for graph: edge extraction, persistence, backlinks, related, traversal | `ExtractEdgesTests`, `ExtractBodyUrlsTests`, `GraphPersistenceTests` | Test file |
| `tests/test_graph_cli.py` | Unit tests for graph CLIs: backlinks/graph resolution, JSON output, error paths | `BacklinksCliTest`, `GraphCliTest` | Test file |
| `tests/test_hubs.py` | Unit tests for hub generation: tag/domain/topic pages, rebuild, human-content preservation, pruning | `BuildHubPagesTest`, `HubsCliTest` | Test file |
| `tests/test_link.py` | Unit tests for link module: URL validation, link operations | Test file |
| `tests/test_render.py` | Unit tests for render module: note rendering, frontmatter generation | Test file |
| `tests/test_reorg.py` | Unit tests for reorg module: bookmark reorganization, folder moves | Test file |
| `tests/test_stats.py` | Unit tests for stats module: statistics collection, metrics | Test file |
| `tests/test_tag_normalize.py` | Unit tests for tag normalization: tag standardization, deduplication | Test file |
| `tests/test_update.py` | Unit tests for update module: bookmark updates, metadata refresh | Test file |
| `tests/test_web_bookmarks.py` | Unit tests for web bookmark integration | Test file |
| `tests/test_web_stats.py` | Unit tests for web statistics integration | Test file |
| `tests/README.md` | Test documentation and guidelines | Documentation |

---

### 4. Data Models & Schemas

#### `NoteProfile` (dataclass, frozen) — `vault_profile.py`
Represents one existing bookmark note parsed from the vault.
```
folder: str            — Relative path under Bookmarks/ (e.g., "ML-AI/LLMs")
title: str             — From frontmatter or filename stem
description: str       — From frontmatter
tags: list[str]        — From frontmatter (parsed bracket list)
parent_topic: str      — From frontmatter
tokens: set[str]       — Normalized searchable tokens from all fields (stopwords removed)
```

#### `BookmarkProfile` (dataclass, frozen) — `vault_profile.py`
Aggregated vault state used by the classifier.
```
notes: list[NoteProfile]                   — All parsed existing notes
folders: list[str]                         — All existing subdirectory paths under Bookmarks/
schema: list[str]                          — Ordered frontmatter field names (union of defaults + observed)
folder_examples: dict[str, list[str]]      — Up to 3 example titles per folder
folder_parent_topics: dict[str, str]       — Most common parent_topic per folder
default_visibility: str                    — Most common visibility value (default: "private")
url_index: dict[str, Path]                 — Normalized URL → note path map for duplicate detection
```

#### `SimilarNote` (dataclass, frozen) — `classify.py`
A scored similarity match between the incoming page and an existing note.
```
folder: str         — Folder of the matched note
tags: list[str]     — Tags of the matched note
parent_topic: str   — Parent topic of the matched note
score: float        — Jaccard-like overlap ratio (tokens intersection / tokens union)
overlap: int        — Raw count of shared tokens
```
**Strong match threshold:** `score >= 0.08 AND overlap >= 2`.

#### `SearchDocument` (dataclass, frozen) — `search_documents.py`
Represents one vault note prepared for FTS5 indexing.
```
path: Path             — Absolute path to the note file
url: str               — From frontmatter
title: str             — From frontmatter or filename stem
folder: str            — Relative path under Bookmarks/ (e.g., "ML-AI/LLMs")
tags: str              — Space-joined tag list
related: str           — Space-joined related-note list
parent_topic: str      — From frontmatter
description: str       — From frontmatter
body: str              — Markdown body text (below frontmatter), preserving headings for chunking
```

#### `CatalogResult` (dataclass, frozen) — `catalog.py`
Result summary for a catalog rebuild operation.
```
database_path: Path
bookmark_count: int
fts_rebuilt: bool
embeddings_rebuilt: bool
embeddings_skipped_reason: str = ""
```

#### `CatalogInfo` (dataclass, frozen) — `catalog.py`
Summary of catalog state for display and doctor checks.
```
database_path: Path
exists: bool
schema_version: int
bookmark_count: int
fts_count: int
embedding_count: int
fetch_log_count: int
chunk_count: int
edge_count: int
job_count: int
```

#### `SearchChunk` (dataclass, frozen) — `chunking.py`
A derived section/chunk record for one bookmark note.
```
path, url, title, folder, tags, related, parent_topic, description
section: str           — Normalized section name (summary, key_ideas, notes, archive, etc.)
chunk_index: int       — Per-note chunk ordinal
chunk_text: str        — Normalized chunk text
token_count: int       — Approximate token count
text_hash: str         — SHA-256 hash of chunk text
```

#### `SearchResult` (dataclass, frozen) — `search_index.py`
A single ranked note or chunk hit returned by `search_index()`, also used as uniform output for semantic/hybrid search.
```
path: Path             — Absolute path to the matched note
url: str               — Bookmark URL
title: str             — Note title
folder: str            — Folder under Bookmarks/
description: str       — Note description
score: float           — Relevance score (BM25, cosine similarity, or RRF fused score)
snippet: str           — Matching chunk/body context snippet
section: str           — Matching section name
chunk_index: int       — Matching chunk ordinal
```

#### `EmbeddingMatch` (dataclass, frozen) — `embeddings.py`
A single cosine-similarity-ranked hit returned by `semantic_search()`.
```
path: Path             — Absolute path to the matched note
url: str               — Bookmark URL
title: str             — Note title
folder: str            — Folder under Bookmarks/
description: str       — Note description
similarity: float      — Cosine similarity score (0.0–1.0; filtered by threshold, default 0.40)
snippet: str           — Matching chunk context
section: str           — Matching section name
chunk_index: int       — Matching chunk ordinal
```

#### `page_data` (dict, ad hoc) — produced by `fetch.py:extract_page_data()`
Now represented by `types.py:PageData` (`TypedDict`).
```python
{
    "url": str,          # Final URL after redirects
    "title": str,        # og:title > <title> tag > domain name
    "description": str,  # meta description or og:description
    "language": str,     # html lang attribute, default "en"
    "content": str,      # Plain text from HTML, truncated to 8000 chars
}
```

#### `BookmarkMetadata` / `NormalizedBookmarkMetadata` (`TypedDict`) — `types.py`
- `BookmarkMetadata`: optional fields from LLM/heuristics before normalization.
- `NormalizedBookmarkMetadata`: required, cleaned metadata passed to `render_note()`.

#### Frontmatter schema (default field order)
```
title, url, type, tags, created, last_updated, language, related, parent_topic, visibility, description
```
`summary` is rendered below the frontmatter block, not inside it.

#### Relationships
- `BookmarkProfile.notes` is a list of `NoteProfile` objects.
- `collect_existing_notes()` builds `BookmarkProfile.url_index`, used by `find_existing_url(url, profile)` for duplicate checks without an extra full note scan.
- `rank_similar_notes()` produces `list[SimilarNote]` from `PageData` + `BookmarkProfile`.
- `normalize_metadata()` consumes `BookmarkMetadata`, `PageData`, `BookmarkProfile`, and `list[SimilarNote]` (+ optional `summary_override`) to produce `NormalizedBookmarkMetadata` passed to `render_note()`.
- `generate_summary()` prefers external `summarize` CLI output, then classifier-provided summary, then direct LLM summarization, and finally `infer_summary()`.

---

### 5. External Dependencies & I/O

#### HTTP Requests (urllib.request)
| Target | Direction | Module | Details |
|--------|-----------|--------|---------|
| Bookmark URL | Read | `fetch.py:fetch_text()` | GET with custom User-Agent; reads up to `MAX_FETCH_BYTES` (1 MB); timeout 20s; uses `urlopen_with_retry()` for transient failure resilience |
| LLM API (`/chat/completions`) | Read/Write | `classify.py:call_llm()` | POST JSON to OpenAI-compatible endpoint for folder/type/tags/related/summary metadata; Bearer token auth; timeout 20s; uses `urlopen_with_retry()` |
| LLM API (`/chat/completions`) | Read/Write | `summarize.py:summarize_with_llm()` | POST JSON to OpenAI-compatible endpoint for summary fallback only when summarize CLI output and classifier summary are unavailable; timeout 180s; uses `urlopen_with_retry()` |
| Embedding API (`/embeddings`) | Read/Write | `embeddings.py:_call_embedding_api()` | POST JSON to OpenAI-compatible endpoint; Bearer token auth; model `text-embedding-3-small` (256 dims); timeout 20s; uses `urlopen_with_retry()` |

#### Local CLI Tools
| Tool | Direction | Module | Details |
|------|-----------|--------|---------|
| `summarize` | Read/Write (subprocess) | `summarize.py:summarize_with_tool()` | Invoked as `summarize <url> --json --plain --no-color --metrics off --stream off --length short --force-summary`; parses JSON `summary` field |

#### File System Operations
| Operation | Location | Module | Details |
|-----------|----------|--------|---------|
| Read `.env` files | Configured via `BOOKMARK_ENV_FILE` or defaults to `$VAULT_PATH/.env` | `paths.py:load_env()` | Parses KEY=VALUE lines into `os.environ` via `setdefault` |
| Read classification guide | Configured via `BOOKMARK_CLASSIFICATION_GUIDE` or bundled default | `classify.py:call_llm()` | Full text included in LLM prompt if file exists |
| Read all `*.md` notes | `$BOOKMARKS_DIR/**/*.md` | `vault_profile.py:collect_existing_notes()` | Single scan to build profile + URL index |
| Duplicate URL lookup | `BookmarkProfile.url_index` | `classify.py:find_existing_url(url, profile)` | Uses prebuilt map (fallback filesystem scan only when profile is not provided) |
| Write new note | `$BOOKMARKS_DIR/<folder>/<slug>.md` | `cli.py:main()` | `mkdir -p` + `write_text()`; skipped in `--dry-run` mode |
| Read notes for search indexing | `$BOOKMARKS_DIR/**/*.md` | `search_documents.py:collect_search_documents()` | Reads frontmatter + body; normalizes into `SearchDocument` records |
| SQLite search database | Configured via `BOOKMARK_SEARCH_INDEX` | `catalog.py`, `search_index.py`, `embeddings.py`, `chunking.py` | Unified catalog DB: chunked FTS5 BM25 search + chunk-level `embedding_store` vectors + `bookmarks` derived metadata + `note_chunks` + `fetch_log` + stubs for edges/jobs |

#### Environment Variables
| Variable | Required | Default | Used In |
|----------|----------|---------|---------|
| `VAULT_PATH` | Yes (for defaults) | None | `paths.py` — determines default paths for bookmarks, search index, guide, and .env |
| `BOOKMARK_LLM_API_KEY` or `OPENAI_API_KEY` or `OPENROUTER_API_KEY` | No (heuristic fallback if absent) | None | `classify.py:get_llm_config()` and `summarize.py:summarize_with_llm()` |
| `BOOKMARK_LLM_MODEL` or `OPENAI_MODEL` or `MODEL_ID` | No | `gpt-4.1-mini` | `classify.py:get_llm_config()` and `summarize.py:summarize_with_llm()` |
| `BOOKMARK_LLM_BASE_URL` or `OPENAI_BASE_URL` | No | Derived by provider | `classify.py:get_llm_config()` and `summarize.py:summarize_with_llm()` |
| `LLM_PROVIDER` | No | empty | `classify.py:get_llm_config()` (if `openrouter` and no explicit base URL, default base URL is `https://openrouter.ai/api/v1`) |
| `BOOKMARKS_DIR` | No | `$VAULT_PATH/Bookmarks` | `paths.py:get_bookmarks_dir()` |
| `BOOKMARK_SEARCH_INDEX` | No | `$VAULT_PATH/Meta/bookmark-search.sqlite3` | `paths.py:get_search_index_path()` |
| `BOOKMARK_CLASSIFICATION_GUIDE` | No | `$VAULT_PATH/Meta/Bookmark-Classification-Guide.md` | `paths.py:get_guide_path()` |
| `BOOKMARK_ENV_FILE` | No | `$VAULT_PATH/.env`, `$VAULT_PATH/../.env` | `paths.py:get_env_paths()` |

If `LLM_PROVIDER` is not `openrouter` and no base URL is set, default base URL is `https://api.openai.com/v1`.

---

### 6. Known Pain Points / Technical Debt

#### Recently resolved

1. **Env var compatibility mismatch** was resolved by supporting `OPENROUTER_API_KEY`, `MODEL_ID`, and `LLM_PROVIDER` in `get_llm_config()`.
2. **Duplicate full note scan** was removed from normal flow: duplicate URL checks now use `BookmarkProfile.url_index`.
3. **Untyped metadata dict flow** was improved by introducing `TypedDict` schemas in `types.py`.
4. **Silent LLM fallback** was improved: failures now print a concise stderr reason before heuristic fallback.
5. **Scattered magic numbers** were reduced via named constants in `classify.py`, `fetch.py`, and `cli.py`.
6. **Minimal helper-only tests** were expanded from 4 to 14 test cases, including summary generation fallback-chain coverage.
7. **Naming inconsistency** across modules was cleaned up: `_coerce_*` → `_normalize_*` in `cli.py`, descriptive constant/function names throughout search and embedding modules.
8. **`_resolve_related()` complexity** was simplified from 4 return paths to a clean 2-branch structure.
9. **YAML scalar serialization** no longer uses JSON quoting; uses plain unquoted scalars via `yaml_scalar()`.
10. **Semantic search noise** was reduced by raising the default similarity threshold to 0.40 and adding a `--threshold` CLI flag.
11. **Vault-coupled paths** were decoupled: all paths are now configurable via environment variables with sensible defaults based on `VAULT_PATH`.
12. **Regex-based HTML parsing** was replaced with `html.parser.HTMLParser` for robust metadata extraction.
13. **No retry/backoff on network calls** was resolved: all HTTP requests now use `urlopen_with_retry()` with exponential backoff and full jitter.
14. **Print-based diagnostics** were replaced with the `logging` module; CLI now supports `--verbose`/`--quiet` flags.
15. **Single-URL-only import** was extended with `--file`/`-f` flag for batch import from files or stdin.
16. **No link health checking** was resolved: `bookmark-check` CLI now validates all bookmarked URLs via HEAD requests.
17. **No integration tests** was resolved: full `build_note()` pipeline tests with mocked network are now in `tests/test_integration.py`.
18. **Fragmented derived state** was consolidated: `catalog.py` provides unified SQLite schema versioning, migrations, and a `bookmarks` derived metadata table, plus stubs for chunks/edges/jobs. All modules (search_index, embeddings, delete, update, reorg, cli) now maintain the catalog on writes. Doctor checks catalog health. `bookmark-rebuild --catalog` rebuilds everything from Markdown.

#### Remaining technical debt

##### High

1. **`validate_folder()` branch complexity:** The function still has multiple remapping branches and should gain targeted branch-by-branch tests.

##### Medium

2. **Test scope remains limited to unit helpers:** Network-behavior tests are still missing (integration tests cover mocked network only).

##### Low

3. **Summary-path coupling:** Summary behavior spans `classify.py`, `summarize.py`, and `render.py` (`infer_summary()`), which increases cross-module coupling and fallback-path complexity.

---
> Source: [mengkeat/bookmark-tools](https://github.com/mengkeat/bookmark-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
