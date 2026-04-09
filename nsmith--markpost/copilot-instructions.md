## markpost

> Markpost is a FastMCP server that syndicates Markdown content to Twitter/X, Threads, and a static blog on S3. It is designed to be called by AI agents via the MCP protocol.

# Claude Code Project Instructions

## Project overview

Markpost is a FastMCP server that syndicates Markdown content to Twitter/X, Threads, and a static blog on S3. It is designed to be called by AI agents via the MCP protocol.

## Commands

```bash
# Run tests
uv run pytest tests/ -v

# Run a single test file
uv run pytest tests/test_formatter.py -v

# Run a single test
uv run pytest tests/test_formatter.py::test_split_by_separator -v

# Start the MCP server (stdio)
uv run python src/markpost/server.py

# Start the MCP server (HTTP)
uv run fastmcp run src/markpost/server.py --transport http --port 9000
```

## Architecture

```
src/markpost/
  server.py          # FastMCP server — tool definitions, orchestration
  config.py          # TOML config loading — frozen dataclasses
  formatter.py       # markdown_to_plain, split_into_thread, markdown_to_html
  publishers/
    twitter.py       # tweepy — sync, posts threads via in_reply_to_tweet_id
    threads.py       # httpx — async, two-step create+publish flow
    blog.py          # boto3 — uploads HTML to S3
```

**Dependency flow:** `server.py` -> `formatter.py` + `publishers/*` -> `config.py`. No circular deps.

**Key constants:** `TWITTER_CHAR_LIMIT = 280` (twitter.py), `THREADS_CHAR_LIMIT = 500` (threads.py).

## Code style and conventions

- `from __future__ import annotations` at the top of every source file
- Type annotations on all function signatures (`list[str]`, `str | None`, `dict`)
- Frozen dataclasses for config (`@dataclass(frozen=True)`)
- MCP tool parameters use `Annotated[type, Field(description="...")]`
- Sync publishers (twitter, blog) are plain functions; async publishers (threads) use `async def`
- `publish_post` is `async` (because it awaits threads); `preview_post` is sync (no I/O)
- Private helpers are prefixed with underscore (`_slugify`, `_split_long_text`, `_split_on_words`)
- HTML output must escape user-provided strings (`html.escape()`)

## Git commits

- Use conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `test:`
- Keep commits atomic — one logical change per commit

## How to add a new feature

Follow TDD. Every feature goes through this loop:

### 1. Write the failing test first

```python
# tests/test_<module>.py
def test_new_behavior():
    from markpost.<module> import new_function
    result = new_function(input)
    assert result == expected
```

### 2. Run the test — confirm it fails

```bash
uv run pytest tests/test_<module>.py::test_new_behavior -v
```

Expected: FAIL (ImportError, AssertionError, etc.). If it passes, your test isn't testing anything new.

### 3. Write the minimal implementation

Only write enough code to make the test pass. No speculative features.

### 4. Run the test — confirm it passes

```bash
uv run pytest tests/test_<module>.py::test_new_behavior -v
```

### 5. Run the full suite — confirm no regressions

```bash
uv run pytest tests/ -v
```

All 27+ tests must pass before committing.

### 6. Commit

```bash
git add <specific files>
git commit -m "feat: description of what was added"
```

## How to add a new platform publisher

1. Add a config dataclass to `config.py` (e.g., `MastodonConfig`) and add it to `MarkpostConfig`
2. Update `load_config()` to parse the new `[mastodon]` section from TOML
3. Create `src/markpost/publishers/mastodon.py` with a `post_to_mastodon(parts, config)` function that returns `list[str]` of post IDs — follow the same signature pattern as twitter.py/threads.py
4. Export the char limit constant (e.g., `MASTODON_CHAR_LIMIT = 500`)
5. Add platform handling in `publish_post` and `preview_post` in `server.py`
6. Update `config.example.toml` with the new section
7. Write tests for each step before implementing

## How to add a new MCP tool

1. Define the function in `server.py` with `@mcp.tool` decorator
2. Use `Annotated[type, Field(description="...")]` for all parameters
3. Write a clear docstring — this becomes the tool description visible to the agent
4. Make it `async` only if it does I/O; keep it sync if it's pure computation
5. Test using `.fn(...)` to call the underlying function:

```python
def test_my_tool():
    from markpost.server import my_tool
    result = my_tool.fn(arg="value")
    assert result == expected
```

## Testing patterns

### Mock external services at the module boundary

```python
# Mock tweepy.Client where it's imported, not where it's defined
with patch("markpost.publishers.twitter.tweepy.Client") as MockClient:
    client = MockClient.return_value
    client.create_tweet.return_value = mock_response
```

### Mock async clients (httpx)

```python
with patch("markpost.publishers.threads.httpx.AsyncClient") as MockClient:
    mock_client = AsyncMock()
    mock_client.post.side_effect = [resp1, resp2]
    MockClient.return_value.__aenter__ = AsyncMock(return_value=mock_client)
    MockClient.return_value.__aexit__ = AsyncMock(return_value=False)
```

### Mock config loading in server tests

```python
@pytest.fixture
def mock_config(tmp_path):
    config_file = tmp_path / "config.toml"
    config_file.write_text("""...""")
    return str(config_file)

async def test_something(mock_config, monkeypatch):
    monkeypatch.setenv("MARKPOST_CONFIG", mock_config)
```

### Testing MCP tool functions

`@mcp.tool` wraps functions in a `FunctionTool` object. Call the underlying function with `.fn(...)`:

```python
result = await publish_post.fn(content="...", platforms=["twitter"])  # async tool
result = preview_post.fn(content="...", platforms=["blog"])           # sync tool
```

### Async tests

Use `@pytest.mark.asyncio` — the `asyncio_mode = "auto"` config in pyproject.toml handles the rest:

```python
@pytest.mark.asyncio
async def test_async_function():
    result = await some_async_function()
    assert result == expected
```

## Things to watch out for

- **Never commit real API keys.** `config.toml` and `.env` are in `.gitignore`. Use `config.example.toml` for documentation.
- **HTML escaping.** Any user-provided string interpolated into HTML must use `html.escape()`. See `formatter.py:markdown_to_html`.
- **FastMCP internals.** Tool introspection uses `mcp._tool_manager._tools` (private API, may change between FastMCP versions). If tests break after a FastMCP upgrade, check this first.
- **Thread splitting edge cases.** The splitter handles: explicit `---` separators, sentence-boundary splits, word-boundary splits, and hard character splits for words longer than the limit. Test all paths when modifying.
- **Threads API is two-step.** Create a container, then publish it. Both calls must succeed. Reply chains use `reply_to_id` referencing the *published* post ID, not the container ID.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nsmith)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nsmith)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
