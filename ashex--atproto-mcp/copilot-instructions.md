## atproto-mcp

> MCP server exposing a semantic-search knowledge base over AT Protocol docs,

# AGENTS.md

MCP server exposing a semantic-search knowledge base over AT Protocol docs,
lexicon schemas, Bluesky API docs, and cookbook examples. Python 3.12+,
FastMCP-style `MCPServer` (`mcp` SDK >=2.0) + txtai hybrid (BM25 + dense)
embeddings, stdio transport.

## Commands

```bash
uv sync                                            # install (uv-managed, hatchling build)
uv run atproto-mcp                                 # run server over stdio
uv run mcp dev src/atproto_mcp/server.py           # MCP Inspector
uv run python -m unittest discover -s tests -p 'test*.py'  # test suite (what CI runs)
uv run python tests/smoke_test.py                  # manual end-to-end check (see gotcha below)
```

- No linter, formatter, or type-checker is configured. Match existing style by hand.
- CI (`.github/workflows/tests.yml`) installs with plain `pip install .` on
  Python 3.12 and runs the `unittest` discover command above.
- `pyproject.toml` requires Python >=3.12 (matches README's "Python 3.12+"
  prerequisite).

## Architecture

Startup flow (`server.py`): `app_lifespan` → `Config.from_env()` → publish
`state.config` → `asyncio.create_task(_warmup(config))` → **yield
immediately**. Nothing slow runs on the startup path: a cold start clones four
repos and downloads a ~130MB model, far longer than a desktop host will wait
during the stdio handshake.

`_warmup()` then, off the startup path:
1. `KnowledgeBase.load()` via `asyncio.to_thread` — **before any network
   access**, so a warm start serves in seconds even offline. On success it
   publishes with `state.commit_kb(kb, generation, save=False)` (already on
   disk, do not rewrite) and sets status `ready`.
2. `fetch_all()` to clone/refresh repos.
3. Cold start: `parse_all()` + `kb.build(save=False)` + `commit_kb()`.
   Warm start: if the loaded index's repo SHAs (`kb.indexed_shas`) or
   embedding pipeline (`kb.indexed_pipeline`) differ from the current config,
   the live index keeps serving while `_rebuild_index()` builds a fresh
   `KnowledgeBase` via `asyncio.to_thread` and commits it.

`_warmup` never raises: exceptions are logged and recorded as status `failed`,
so the server stays connected and can explain itself. It then **retries with
exponential backoff** (`_RETRY_INITIAL_DELAY` → `_RETRY_MAX_DELAY`, both
module-level so tests can zero them) for as long as `state.kb is None` — the
common failure is a desktop host launching the server before the network is
up. Retrying stops once an index is serving; `refresh_sources` remains the
explicit recovery path and resets the status to `ready`.

`tests/test_warmup.py`'s `_StateIsolationMixin` zeroes both delays for every
test in the file — without that, any test that fails a cold start spins in the
retry loop and stalls the suite.

**Warmup status** (`state.py`): `set_status()` / `get_status()` /
`status_message()` over the `STATUS_*` constants
(`starting|fetching|indexing|ready|failed`). `get_kb()` raises
`KnowledgeBaseNotReady` (a `RuntimeError` subclass) carrying
`status_message()` when no index is in service. `tools.py` and `resources.py`
each define a local `_reports_warmup` decorator, applied under `@mcp.tool()` /
`@mcp.resource()`, that turns that exception into a normal response — plain
text for tools, a `{"status", "message"}` JSON object for resources. It uses
`functools.wraps` because the generated MCP schema reads the docstring and
signature; do not replace it with a plain wrapper. Progress statuses are set
only when **no** index is serving — a background refresh behind a live index
must stay `ready`.

**Rebuild concurrency** — two separate mechanisms, do not confuse them:

1. `state.index_build_lock` serializes the *work*. `fetch_all()` rewrites the
   repo clones, `parse_all()` reads them, and `KnowledgeBase.save()` rewrites
   `<cache>/index/` in place, so two overlapping builds would corrupt each
   other. Because `parse_all` swallows per-source exceptions, the damage is
   silent: a source whose tree was deleted mid-parse yields zero chunks, and
   the truncated index is saved stamped with *fresh* SHAs, which nothing ever
   invalidates. **Any new code path that fetches, parses, or saves must take
   this lock.** It is a plain `threading.Lock` — not reentrant, so never call
   `_rebuild_index` while holding it (`_warm_index` returns a "needs rebuild"
   flag instead of calling it directly). `refresh_sources` acquires it
   `blocking=False` and declines with a message rather than hanging the tool
   call for minutes.
2. The **generation token** guards *publication* only. Every rebuild claims
   one via `state.claim_generation()` before starting; `state.commit_kb(kb,
   generation)` saves to disk and swaps `state.kb` inside one locked critical
   section, discarding results whose generation is older than the one already
   published — a slow rebuild can never overwrite a newer index in memory or
   on disk. Always check the return value: `False` means your result was
   thrown away, so do not then log success or set status `ready`.

| Module | Role |
| --- | --- |
| `config.py` | `Config` (frozen dataclass, env-driven) + `REPOS` source registry + `SOURCE_*` constants |
| `fetcher.py` | GitPython shallow clones into `<cache>/repos/`; SHA/timestamp meta in `<cache>/meta/*.json`; staleness check vs `refresh_hours` |
| `parser.py` | `ContentChunk` dataclass; per-source parsers (`parse_atproto_website`, `parse_bsky_docs`, `parse_lexicons`, `parse_cookbook`) → `parse_all` |
| `indexer.py` | `KnowledgeBase` wrapping txtai `Embeddings`; build/load/save; search with tag/source filtering |
| `state.py` | Module-level `kb`/`config` globals set during lifespan; access via `get_kb()` / `get_config()`; rebuild generation tokens; warmup status API |
| `tools.py`, `resources.py`, `prompts.py` | `register_*(mcp)` functions that define everything as inner closures decorated with `@mcp.tool()` etc. |
| `manifest.json`, `.mcpbignore` | MCPB bundle metadata (see "MCPB bundle" below); packed from the repo root |

### Key patterns

- **Shared state is module-level**, not passed through MCP server context. Tools
  call `state.get_kb()`, which raises `KnowledgeBaseNotReady` (a `RuntimeError`
  subclass) whenever no index is in service — before lifespan runs, during
  warmup, or after a failed warmup.
- **Registration pattern**: all tools/resources/prompts are closures inside
  `register_tools(mcp)` / `register_resources(mcp)` / `register_prompts(mcp)`.
  Add new ones there; docstrings + type hints become the MCP schema, so write
  agent-facing docstrings with `Args:` sections like the existing ones.
- **Tools return formatted strings**, not JSON (except resources, which return
  JSON). Not-found paths return "Did you mean" suggestions.
- **Logging goes to stderr only** — stdout is the MCP JSON-RPC channel.
  Never `print()` in server code.

## The tag/search system (most gotcha-dense area)

- Tags are `key:value` strings (`source:lexicons`, `content_type:guide`,
  `topic:specs`, `namespace:app.bsky.feed`, `language:python`,
  `lexicon_type:record`). Built per-source by `_build_*_tags()` in `parser.py`.
- `encode_tags()` stores them in txtai as a **sorted, deduped, pipe-delimited
  string** (`|a|b|c|`) so SQL `LIKE '%|tag|%'` matches are unambiguous
  (prevents `source:lexicons` matching `source:lexicons-extra`). Keep the
  leading/trailing pipes if you touch this.
- Filtered search (`KnowledgeBase._filtered_search`) builds a txtai SQL query
  with `similar(:query, N)` + `tags LIKE :tagN` bind-parameter clauses
  (values passed via `parameters=`; no manual escaping). The over-fetch
  candidate count `N` (`max(limit*5, 100)`) **must be inside `similar()`**
  and the SQL `LIMIT` must match it — txtai caps results at the SQL LIMIT
  and ignores the Python-side `limit` kwarg when SQL has a LIMIT. The final
  trim to `limit` happens in Python after `_enrich_results`, which
  **re-applies the source filter on chunk metadata** as a second line of
  defense — the regression test `test_source_filtered_search_regression.py`
  guards this behavior, and `FilteredSearchSqlTests` in
  `test_freshness_and_retrieval.py` guards the SQL shape.
- `document_text()` in `indexer.py` prepends a label line (title | source |
  topic/namespace/content_type/lexicon_type/language tag values) to each
  indexed document so labels participate in BM25 and dense matching. Search
  result `text` therefore starts with this label line.
- Chunk `uid`: `{source}:{nsid}` for lexicons, else
  `{source}:{file_path}:{title}`. Changing uid format invalidates cached
  indexes silently.

## Index persistence

- txtai index lives in `<cache>/index/`; chunk metadata is saved separately
  as `<cache>/index/chunk_meta.json` (`_save_chunk_meta`/`_load_chunk_meta`),
  and the repo SHAs + embedding pipeline config the index was built from
  live in `<cache>/index/build_meta.json` (exposed as `kb.indexed_shas` /
  `kb.indexed_pipeline` — these drive background staleness rebuilds; the
  pipeline check matters because txtai `load()` restores the saved pipeline
  and silently ignores constructor args).
  If you add a `ContentChunk` field that must survive restarts, you must add
  it to both save and load paths (and `_META_KEYS` in `indexer.py`).
- `kb.build(chunks, shas=…, save=False)` builds in memory only; publish via
  `state.commit_kb(kb, generation)`, which saves before swapping. Always pass
  `shas=get_cached_shas(config)` when building, or the next startup will
  consider the index stale. Use `commit_kb(…, save=False)` **only** for an
  index just loaded from disk — anything freshly built must be persisted.
- On reload, `ContentChunk.text` is empty (full text lives in txtai);
  lexicon `raw_json` is the exception, persisted in metadata so
  `get_lexicon` / resources can return the full schema.
- `_load_chunk_meta` reconstructs a `source:` tag when tags are missing —
  backward compat for pre-tag indexes; keep such compat shims when changing
  the metadata format.
- **No automatic cache invalidation on format changes.** Delete
  `~/.cache/atproto-mcp/index/` (or set `ATPROTO_MCP_CACHE_DIR` to a temp
  dir) after changing parsing/indexing logic, or you'll test against stale
  data.

## Testing conventions

- Framework: stdlib `unittest`, files named `test*.py` in `tests/`.
- Tests **never touch txtai or the network**: they inject a `_FakeEmbeddings`
  directly into `kb._embeddings` and populate `kb._chunks_by_uid` by hand
  (see `test_source_filtered_search_regression.py`). The fake parses the
  generated SQL's `tags LIKE '%|…|%'` clauses — if you change the SQL shape
  in `_filtered_search`, update the fakes too. Filesystem-dependent parser
  tests use `tempfile` dirs with `Config(cache_dir=Path(tmp))`
  (see `test_freshness_and_retrieval.py`).
- `tests/smoke_test.py` is deliberately named so `unittest` discovery skips
  it: it's a manual script that clones real repos, downloads the ~130MB
  sentence-transformer model, and builds a full index. Run it only for
  end-to-end verification, ideally with `ATPROTO_MCP_CACHE_DIR=/tmp/...`.
- To enumerate registered tools/prompts without touching MCP SDK internals,
  register against a fake server object exposing `.tool()` / `.prompt()` /
  `.resource()` that return identity decorators (see `test_manifest.py`'s
  `_Recorder`). Everything is an inner closure, so this is the only stable
  way to reach them.
- `test_warmup.py` patches `server.fetch_all` / `parse_all` /
  `get_cached_shas` / `current_pipeline` and `KnowledgeBase.load` / `build` /
  `save` — patch on the `server` module, not the defining module, since
  `server.py` imports the names directly.

## MCPB bundle

- The bundle is packed **from the repo root** (`mcpb pack .`): `manifest.json`
  + `pyproject.toml` + `src/` only, ~32KB. `.mcpbignore` extends the CLI's
  built-in exclusions; keep `dist/` in it — the release workflow runs
  `uv build` before `mcpb pack`, so the wheel would otherwise land inside
  the bundle.
- `server.type` is `"uv"`, which **requires** `manifest_version: "0.4"`. The
  host installs dependencies from `pyproject.toml`; never add `server/lib/`
  or a bundled venv.
- `mcp_config` runs the `atproto-mcp` console script
  (`uv run --directory ${__dirname} atproto-mcp`), not `server.py` directly,
  so the bundle and the PyPI install exercise identical startup code.
- `manifest.json` duplicates the tool roster/descriptions and prompt
  templates. **`tests/test_manifest.py` fails on any drift**, including
  byte-level prompt text: it re-derives each template by calling the Python
  prompt with a sentinel and substituting `${arguments.<name>}`. Change the
  Python, then update the manifest to match.
- The manifest `version` is **not** asserted in the test suite — it only has
  to match at release, and asserting it there makes every working tree fail
  mid-bump. `release.yml`'s first step checks the tag, `pyproject.toml`, and
  `manifest.json` all agree, before the tests and before anything is
  published.
- MCPB requires a static `text` on every declared prompt, so all four prompts
  in `prompts.py` must stay pure string templates. If one ever needs to call
  the knowledge base, drop it from the manifest's `prompts[]` and set
  `prompts_generated: true` instead.
- Validate with `mcpb validate .` before packing. Do **not** gate anything on
  `mcpb verify`: node-forge does not implement PKCS#7 verification, so CLI
  2.1.2 reports every signed bundle as unsigned.
- Adding/renaming a tool means updating `tools.py`, the manifest `tools[]`
  array, the README table, and `POWER.md`.

## Fetcher gotchas

- The `atproto` monorepo is cloned via **sparse checkout of `lexicons/` only**
  (`sparse_paths` in `REPOS`); all clones are `depth=1`. Updates do
  `fetch --depth=1` + `reset --hard origin/<branch>`, and fall back to
  rm-and-reclone on git errors.
- Note the naming mismatch: the repo key is `atproto`/`atproto-website`, but
  the chunk source constant for lexicons is `"lexicons"`
  (`SOURCE_LEXICONS`), not the repo name.
- Adding a data source = add entry to `REPOS`, a `SOURCE_*` constant, a
  `parse_<source>()` + `_build_<source>_tags()` in `parser.py`, wire into
  `parse_all`, and extend `_VALID_SOURCES` in `tools.py`.

## Parser gotchas

- **atproto-website locale layout**: content lives at
  `src/app/[locale]/<section>/<slug>/{en,ja,ko}.mdx`. `parse_atproto_website`
  skips non-`en` locale-named files, derives the title from the slug
  directory when the stem is a locale code, and `_extract_path_topic` /
  `_atproto_website_url` strip the literal `[locale]/` segment plus
  trailing `/en`.
- `_chunk_by_headings` caps chunks at `max_chunk_len` (default 1200 chars),
  splitting oversized sections on paragraph boundaries into
  `"Title > Heading (part n)"` chunks.
- **Cookbook chunks come in two kinds**: the project chunk
  (`file_path == project_name`, README + file listing) and per-source-file
  chunks (`file_path == "project/file.py"`). `list_cookbook_examples` and
  friends distinguish them by `"/" in file_path` — preserve that invariant.

## Environment variables

`ATPROTO_MCP_CACHE_DIR` (default `~/.cache/atproto-mcp`),
`ATPROTO_MCP_REFRESH_HOURS` (default 24), `ATPROTO_MCP_EMBEDDING_MODEL`
(default `BAAI/bge-small-en-v1.5`). Parsed in
`Config.from_env()`; invalid values fall back to defaults silently.

## Style notes

- Full type hints everywhere, `from __future__` not used — modern syntax
  (`X | None`, `Self`, builtin generics) directly.
- Google-style docstrings with `Args:`/`Returns:`; module docstrings on every
  file.
- Private helpers prefixed `_`; `logger = logging.getLogger(__name__)` per
  module.
- If you add/rename tools, env vars, or sources, update `README.md` tables
  and `POWER.md` (Kiro power onboarding doc) to match.

---
> Source: [Ashex/atproto-mcp](https://github.com/Ashex/atproto-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
