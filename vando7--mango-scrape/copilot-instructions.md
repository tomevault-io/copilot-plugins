## mango-scrape

> Personal-use "deep dive" scraper for local LLMs. Takes URLs, returns clean markdown + metadata. Intended workflow: a local model gets search snippets, then calls `deep_dive` on results it wants to read in full.

# AGENTS.md

Personal-use "deep dive" scraper for local LLMs. Takes URLs, returns clean markdown + metadata. Intended workflow: a local model gets search snippets, then calls `deep_dive` on results it wants to read in full.

## Architecture

```
LM Studio
  │  spawns via stdio
  ▼
mcp_shim.py (thin, self-bootstraps) ──HTTP──▶ localhost:8765
                                                  │
                                                  ▼
                                            server.py (FastAPI)
                                                  │
                                                  ▼
                                            scraper.py (patchright + trafilatura)
```

The shim auto-starts the FastAPI server on port 8765 if it isn't already running. All deps live in `.venv`. The shim is intentionally thin: stdio MCP in, HTTP POST out.

## Files

| Path | Role |
|---|---|
| `scraper.py` | Scraping logic. `_compact`, `launch_browser`, `_scroll_page`, `_extract_links`, `_extract_reddit_comments`, `scrape_one`, `scrape_many`, `screenshot_one/many`, `download_file`, `clone_repo`, `list_files`, `cat_file`, `hn_search`, `reddit_search`. YouTube: delegates transcript to `yt_transcript`. Reddit URL scraping: www→old→m.reddit fallback chain (www first for SSR post body). Post-body detection JS check before committing to a URL. Improved comment selectors for Reddit's current DOM. |
| `server.py` | FastAPI wrapper. Lifespan owns a shared browser. Endpoints: `POST /scrape`, `/screenshot`, `/scrape_youtube`, `/transcript_page`, `/download`, `/clone`, `/list`, `/cat`, `/search`, `/hn_search`, `/reddit_search`, plus `GET /health`. Brave search via `BRAVE_API_KEY`. Idle shutdown after `DEEP_DIVE_IDLE_TIMEOUT` seconds (default 300). Writes `server.pid` on startup. Listens on `8765`. |
| `yt_transcript.py` | YouTube transcript fetching with SQLite cache and word-based pagination (500-word pages). Functions: `get_transcript`, `paginate_transcript`, `get_page`, `cache_get`, `cache_put`. Cache path defaults to `./cache/yt_transcripts.db` next to the file (overridable via `DEEP_DIVE_CACHE_DB`). |
| `shim/mcp_shim.py` | stdio MCP server. `_fmt` formats a single dict as flat key=value lines; `_fmt_list` formats `{status, query, num_results, results}` shapes with `===` between blocks. Multi-line values keep real newlines — no escape artifacts. `_post()` does the HTTP call with one auto-restart-and-retry on `ConnectError` (raises `ServiceError` on persistent failure). `_ensure_server()` only auto-starts when the target host is local. |
| `shim/pyproject.toml` | Shim deps: `mcp`, `httpx`. |
| `pyproject.toml` | Host-side dev deps (for running `server.py`). |
| `README.md` | Human-facing setup. |

## Commands

```bash
# One-shot: shim auto-starts the server on first call
uv run python shim/mcp_shim.py

# Manual: start server only (shim will do it automatically anyway)
cd C:\software\searchmcp\scraper\scrape
uv sync                          # installs deps + managed Python into .venv
uv run patchright install chromium  # downloads patched Chromium binary
uv run uvicorn server:app --port 8765

# Smoke tests (server must be running)
curl -s http://localhost:8765/health
curl -s -X POST http://localhost:8765/scrape -H 'Content-Type: application/json' \
  -d '{"urls":["https://example.com"]}' | jq .

# Stop (if started manually)
taskkill /F /IM uv.exe
```

## Conventions and gotchas

**Stdio MCP hygiene (shim).** The shim speaks JSON-RPC over stdout. Never `print()` in `mcp_shim.py` or anything it imports. All logging to stderr (`logging.basicConfig(stream=sys.stderr, ...)`). A stray stdout byte corrupts the protocol and the client closes the connection silently.

**Use patchright, not playwright.** Import is `from patchright.async_api import …`. Patchright patches the `Runtime.enable` CDP leak that regular Playwright (and `playwright-stealth`) can't mask. Chromium-only. Do not "upgrade" to vanilla Playwright — you lose stealth.

**Patchright's Chromium is separate.** `pip install patchright` does not fetch a browser. `patchright install chromium` downloads a patched build distinct from any existing Playwright browser. Run once: `uv run patchright install chromium`.

**Shared browser, fresh context per URL.** `server.py` launches one browser in `lifespan` and reuses it. Each scrape creates a new `browser.new_context()` (cheap) and closes it. Do not launch per request (1–2 s overhead). Do not reuse contexts (leaks cookies/state).

**Errors are per-URL, never raise.** `scrape_one` catches everything and returns `{status: "error", error: ...}`. `scrape_many` uses `asyncio.gather(..., return_exceptions=True)`. Callers always get one result per input URL — preserve this contract.

**Self-bootstrapping shim with auto-restart.** The shim checks `{SERVICE_URL}/health` on startup; if down, it launches uvicorn as a subprocess and waits up to 30s for readiness. The same path runs again from `_post()` whenever a tool call hits `httpx.ConnectError` — so when the server idle-shuts (`DEEP_DIVE_IDLE_TIMEOUT`), the next tool call transparently respawns it and retries once. Auto-start is skipped when `DEEP_DIVE_URL` points to a non-local host. Persistent failures surface as `ServiceError` and tools return per-tool error dicts instead of hanging.

**LM Studio mcp.json.** Located at `%USERPROFILE%\.lmstudio\mcp.json` on Windows. Follows Cursor's notation: `{"mcpServers": {...}}`. LM Studio spawns the shim via stdio — no env vars needed, server auto-starts. Example:
```json
{
  "mcpServers": {
    "deep-dive": {
      "command": "C:\\software\\searchmcp\\scraper\\scrape\\.venv\\Scripts\\python.exe",
      "args": ["shim/mcp_shim.py"]
    }
  }
}
```
Per-server stderr is visible in the Program tab.

**Screenshots return MCP ImageContent or TextContent.** `deep_dive_screenshot` forwards screenshot b64 from the server into `mcp.types.ImageContent(type="image", data=<b64>, mimeType="image/png")` blocks — the model sees actual images. On per-URL failure or service-level error, the corresponding block is a `TextContent` with the error message instead, so the model can read what went wrong rather than a blank/fake image.

**YouTube transcript pagination.** Long transcripts are split into ~500-word pages. `get_youtube_transcript` returns page 1 + metadata (`total_pages`, `page_num`). The model is told to say "next" for more pages. Use `deep_dive_transcript_page(video_id, page_num)` to fetch subsequent pages. Transcripts are cached in SQLite (`./cache/yt_transcripts.db`) keyed by `(video_id, language)` — second call hits cache instantly.

**Output format and divider.** Flat key=value lines, no JSON or brackets, real newlines preserved. Multi-line values continue on subsequent lines until the next `key=` or the `===` divider. The `===` divider separates per-URL or per-result blocks (chosen over `---` to avoid collisions with markdown horizontal rules in scraped content).

**Web search.** `POST /search` calls Brave Search (`https://api.search.brave.com/res/v1/web/search`) when `BRAVE_API_KEY` is set. Shim exposes `web_search(query, num_results, language, deep)`. With `deep=True` the shim follows up with a single `/scrape` call for all result URLs (concurrent inside `scrape_many`) and merges the markdown into each result's block. Default off — model is expected to triage snippets and call `deep_dive` on URLs it actually wants. Response: flat key=value lines with url/title/snippet per result, plus the merged scrape fields when deep.

**HN and Reddit search.** `POST /hn_search` hits the Algolia HN Search API (free, no auth). `POST /reddit_search` hits Reddit's public `search.json` with a realistic User-Agent. Both are pure-HTTP — no browser, no patchright. Reddit aggressively rate-limits unauthenticated calls; the convention is to scope the query to a subreddit on 429/403 rather than retry blindly. These are search/triage tools — the model should still call `deep_dive` on the URLs it wants the full text of.

**Reddit scraping.** For `reddit.com` URLs (not old.reddit or m.reddit), scraper.py tries: **www.reddit first** (has SSR post body), then old.reddit (cleaner HTML, no JS), then m.reddit. `_inject_stealth()` runs before navigation to override `navigator.webdriver`, `window.chrome`, plugins, languages, and hardwareConcurrency — bypassing old.reddit's bot detection. Realistic headers (`Accept`, `Sec-Fetch-*`, etc.) are set via context-level `extra_http_headers`. On www.reddit.com it waits longer for network idle (`timeout_s * 500ms`) and scrolls the page to trigger lazy-loaded comments. Before committing to a URL, a JS check verifies the post body/comments actually exist in the DOM — if not (old.reddit often just has the shell), it tries the next URL. trafilatura is called with `include_comments=True`. The `_extract_reddit_comments()` JS function attempts DOM extraction as a fallback with improved selectors for Reddit's current DOM. On www.reddit.com it clicks "Load more comments" buttons up to 10 times plus 5 scroll cycles to maximize comment capture. m.reddit.com works but loads fewer comments.

## House style

- Match the terse register of the existing files: no docstrings on obvious functions, no decorative type annotations, no speculative abstractions, no feature flags.
- Three similar lines beat a premature abstraction.
- Keep the shim thin. If a feature can live in the server, put it there.
- Log at INFO sparingly — one line per operation. Tracebacks at WARNING for handled errors, ERROR for unhandled.
- Don't add error handling for conditions that can't happen. Validate at the HTTP and MCP boundaries only.

---
> Source: [Vando7/mango-scrape](https://github.com/Vando7/mango-scrape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
