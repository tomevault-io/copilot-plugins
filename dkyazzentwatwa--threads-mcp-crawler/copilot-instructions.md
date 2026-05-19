## threads-mcp-crawler

> Internal documentation for developers working on `threads_scraper_mcp`.

# Threads Scraper MCP - Developer Guide

Internal documentation for developers working on `threads_scraper_mcp`.

## Architecture Overview

### Design Philosophy

1. **Robustness over cleverness**: Dual parsing (JSON + HTML) ensures resilience
2. **Centralized selectors**: All CSS selectors in one `ThreadsSelectors` class
3. **Graceful degradation**: Return partial data rather than failing completely
4. **Ethical scraping**: Built-in throttling and caching, no circumvention attempts
5. **Proven patterns**: Architecture mirrors successful `substack_mcp` implementation

### Tech Stack

- **Python 3.9+**: Modern type hints (`|` union, `Optional`, generics)
- **Pydantic v2**: Type-safe data validation
- **Playwright**: Headless Chrome browser automation (bypasses login wall)
- **BeautifulSoup + lxml**: HTML parsing (fallback)
- **VADER**: Sentiment analysis for social media
- **MCP SDK**: Model Context Protocol for AI integration
- **cachetools**: Thread-safe TTL caching

## Module Responsibilities

### `settings.py`
- Environment-based configuration via `@dataclass`
- `RuntimeSettings.from_env()` factory pattern
- Global `SETTINGS` instance
- **Env vars**: `THREADS_THROTTLE`, `THREADS_CACHE_TTL`, `THREADS_SESSION_COOKIE`, etc.

### `cache.py`
- **Copied verbatim from substack_mcp** (proven implementation)
- Thread-safe `TTLCache` with `RLock`
- `@cached` decorator for function memoization
- `clear_cache(prefix)` for selective invalidation
- 256-item limit, 10-minute TTL default

### `models.py`
- Pydantic v2 models for type safety
- **Core models**: `Profile`, `Thread`, `Reply`, `EngagementMetrics`
- **Analytics models**: `SentimentBreakdown`, `KeywordScore`, `ProfileAnalytics`
- All fields use `Optional` for graceful degradation
- `raw: Dict[str, Any]` field preserves original data for debugging

### `parsers.py`
- **CRITICAL FILE**: Most likely to need updates when Threads changes HTML
- `ThreadsSelectors` class: Centralized CSS selectors
- **Dual parsing system**:
  1. Primary: JSON from `__NEXT_DATA__` script tag
  2. Fallback: HTML selectors
- Helper functions: `_extract_next_data()`, `_parse_datetime()`, `_parse_engagement_from_json()`, etc.
- Main parsers: `parse_profile()`, `parse_thread()`, `parse_profile_threads()`, `parse_replies()`, `parse_search_results()`

### `client.py`
- `ThreadsPublicClient` class with **Playwright headless browser**
- **GraphQL interception**: Captures API responses from browser network
- **Throttling**: Timestamp-based, enforced in `_throttle()`
- **Lazy browser init**: Browser only starts on first request
- **@cached methods**: All fetch methods use decorator
- Error handling: Return `None`/empty list gracefully
- Methods: `get_profile()`, `get_profile_threads()`, `get_thread()`, `get_thread_replies()`, `search_threads()`
- **Key method**: `_fetch_with_graphql()` - navigates page and intercepts GraphQL responses

### `analysis.py`
- Analytics adapted for **short-form content**
- **VADER sentiment**: Compound score + pos/neg/neutral breakdown
- **Keyword extraction**: TF-based with stopword filtering
- **Hashtag extraction**: Separate regex capture, merged into keywords
- **Posting cadence**: Average days between posts
- **Engagement averages**: Mean likes/replies/reposts/quotes
- Returns `ProfileAnalytics` with raw data + computed metrics

### `mcp_server.py`
- MCP server setup with `Server("threads-scraper-mcp")`
- **6 tool handlers**: `get_profile`, `get_profile_threads`, `get_thread`, `get_thread_replies`, `search_threads`, `analyze_profile_threads`
- **Async/sync bridge**: `run_in_executor()` pattern
- **Error handling**: Try/catch with structured JSON errors
- **MCP bug workaround**: `create_text_result()` helper (serialization issue in MCP >1.10.0)

## Core Patterns

### 1. Dual Parsing System

**Why**: Threads uses React/NextJS with `__NEXT_DATA__` JSON blob for server-side rendering. This is more stable than HTML classes.

```python
def parse_profile(handle: str, html: str) -> Optional[Profile]:
    soup = BeautifulSoup(html, "lxml")
    next_data = _extract_next_data(soup)  # Try JSON first

    if next_data:
        user_data = next_data.get("props", {}).get("pageProps", {}).get("user")
        if user_data:
            return Profile(...)  # Structured data

    # Fallback to HTML
    name_elem = soup.find("h1")
    return Profile(display_name=name_elem.text, ...)
```

**Update strategy**:
1. When parsing fails, inspect `__NEXT_DATA__` JSON structure in browser DevTools
2. Update JSON path navigation in parsers
3. Only update HTML selectors if JSON structure completely changed

### 2. Centralized Selectors

**Why**: CSS classes change frequently. Centralizing makes updates fast.

```python
class ThreadsSelectors:
    """UPDATE HERE when HTML changes"""
    PROFILE_NAME = "h1[role='heading']"
    THREAD_TEXT = '[data-testid="thread-text"]'
    LIKE_COUNT = '[aria-label*="like"]'
```

**Update workflow**:
1. Identify broken selector (test failure or user report)
2. Inspect element in browser DevTools
3. Update `ThreadsSelectors` constant
4. No changes needed in parsing logic

### 3. Throttling Pattern

**Why**: Prevent rate limiting, respect ToS.

```python
def _throttle(self) -> None:
    delta = time.time() - self._last_request_ts
    remaining = self.settings.throttle_seconds - delta
    if remaining > 0:
        time.sleep(remaining)
    self._last_request_ts = time.time()
```

**Critical**: Called in `_get()` before every HTTP request. Non-negotiable.

### 4. Caching Pattern

**Why**: Reduce repeat requests, improve response time.

```python
@cached
def get_profile(self, handle: str) -> Optional[Profile]:
    ...
```

**Cache invalidation**:
```python
from threads_scraper_mcp.cache import clear_cache
clear_cache("ThreadsPublicClient.get_profile")  # Specific method
clear_cache()  # All cache
```

### 5. Async/Sync Bridge

**Why**: MCP handlers are async, but HTTP client is sync.

```python
@server.call_tool()
async def handle_call_tool(name: str, arguments: dict):
    loop = asyncio.get_event_loop()
    profile = await loop.run_in_executor(
        executor, threads_client.get_profile, handle
    )
```

**Critical**: Use `ThreadPoolExecutor`, not thread-per-request.

## Development Workflow

### Setup

```bash
# Clone repo
cd threads-mcp-crawler

# Create venv
python3 -m venv .venv
source .venv/bin/activate

# Install in editable mode
pip install -e .

# Or install requirements directly
pip install -r requirements.txt

# IMPORTANT: Install Playwright browser
playwright install chromium
```

### Running Locally

```bash
# Direct execution
python run_mcp_server.py

# With environment vars
THREADS_THROTTLE=1.0 THREADS_CACHE_TTL=300 python run_mcp_server.py
```

### Testing with Claude Code

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "threads-scraper-dev": {
      "command": "/path/to/threads-mcp-crawler/.venv/bin/python3",
      "args": ["/path/to/threads-mcp-crawler/run_mcp_server.py"],
      "env": {
        "THREADS_THROTTLE": "1.0",
        "THREADS_SESSION_COOKIE": "your_session_cookie_here"
      }
    }
  }
}
```

Replace `/path/to/threads-mcp-crawler` with your actual installation path. Restart Claude Desktop to pick up changes.

### Manual Testing

```python
# Test profile fetching
from threads_scraper_mcp.client import ThreadsPublicClient

client = ThreadsPublicClient()
profile = client.get_profile("zuck")
print(profile.model_dump_json(indent=2))

# Test thread fetching
threads = client.get_profile_threads("zuck", limit=5)
for thread in threads:
    print(f"{thread.id}: {thread.text[:50]}...")

# Test analytics
from threads_scraper_mcp.analysis import analyze_profile_threads
analytics = analyze_profile_threads(profile, threads)
print(analytics.sentiment)
print(analytics.keywords[:5])
```

## Common Tasks

### 1. Update Selectors (HTML Changed)

**Symptom**: Parsing returns `None` or partial data

**Steps**:
1. Visit threads.net in browser
2. Open DevTools → Elements
3. Find broken element, note new selector
4. Update `ThreadsSelectors` in `parsers.py`:
   ```python
   class ThreadsSelectors:
       PROFILE_NAME = "h1.new-class-name"  # <- Update here
   ```
5. Test with `client.get_profile("little_hakr")`

### 2. Update JSON Paths (API Changed)

**Symptom**: JSON extraction returns empty data

**Steps**:
1. Visit threads.net page
2. View source → find `<script id="__NEXT_DATA__">`
3. Copy JSON to formatter/viewer
4. Identify new path to data
5. Update parser:
   ```python
   user_data = props.get("userData")  # Old
   user_data = props.get("user", {}).get("profile")  # New
   ```

### 3. Add New Field to Model

**Steps**:
1. Update `models.py`:
   ```python
   class Profile(BaseModel):
       ...
       new_field: Optional[str] = None  # Add field
   ```
2. Update parser:
   ```python
   new_field=user_data.get("new_field_key")
   ```
3. Test round-trip

### 4. Add New MCP Tool

**Steps**:
1. Add client method in `client.py`:
   ```python
   @cached
   def new_method(self, arg: str) -> Result:
       ...
   ```
2. Add tool definition in `mcp_server.py`:
   ```python
   Tool(
       name="new_tool",
       description="...",
       inputSchema={...}
   )
   ```
3. Add handler:
   ```python
   elif name == "new_tool":
       result = await loop.run_in_executor(...)
       return create_text_result(...)
   ```

### 5. Adjust Throttling/Caching

**Throttling** (slower requests):
```bash
export THREADS_THROTTLE=5.0  # 5 seconds between requests
```

**Caching** (longer TTL):
```bash
export THREADS_CACHE_TTL=1800  # 30 minutes
```

**Clear cache programmatically**:
```python
from threads_scraper_mcp.cache import clear_cache
clear_cache()  # Full clear
```

## Known Issues & Workarounds

### Issue: MCP Serialization Bug (>1.10.0)

**Symptom**: Tool responses fail with serialization error

**Workaround**: Lock MCP to 1.10.0 in `requirements.txt`:
```
mcp==1.10.0
```

**Helper function**:
```python
def create_text_result(text: str, is_error: bool = False):
    return [types.TextContent(type="text", text=text)]
```

### Issue: Rate Limiting (HTTP 429)

**Symptom**: Requests fail with 429 status

**Workaround**:
1. Increase `THREADS_THROTTLE` to 5-10 seconds
2. Reduce request volume
3. Wait 15-30 minutes before retrying
4. Consider session cookie (may help)

### Issue: Private Profiles Return None

**Expected behavior**: Not a bug

**Workaround**: Check `is_private` field if profile data available, otherwise return clear error message.

### Issue: Search Returns Empty

**Symptom**: `search_threads()` returns `[]`

**Causes**:
1. Public search disabled (requires auth)
2. No results for query
3. Rate limited

**Workaround**: Provide `THREADS_SESSION_COOKIE`

### Issue: Pagination Limited

**Symptom**: Only ~20 threads returned from profile

**Cause**: Initial page load limitation

**Future enhancement**: Implement pagination via API endpoints or infinite scroll simulation

## Performance Considerations

### Bottlenecks

1. **HTTP requests**: 2-second throttle means 30 requests/minute max
2. **HTML parsing**: BeautifulSoup with lxml is fast enough (milliseconds)
3. **Sentiment analysis**: VADER is lightweight (milliseconds per thread)
4. **Cache lookups**: O(1) dict lookup with thread lock (negligible)

### Optimization Tips

1. **Use caching**: Don't clear cache unnecessarily
2. **Batch operations**: Fetch multiple profiles in parallel (respecting throttle)
3. **Limit results**: Only fetch what you need (`limit` parameter)
4. **Session reuse**: Keep `ThreadsPublicClient` instance alive

### Scaling Limits

- **Single client**: ~30 requests/minute (throttle-limited)
- **Cache size**: 256 items max (LRU eviction)
- **Memory**: ~10MB for typical usage, ~100MB with full cache

## Security & Privacy

### Session Cookies

**Risk**: Session cookies provide authenticated access.

**Mitigation**:
1. Never log session cookies
2. Never commit to version control (`.gitignore`)
3. Use environment variables only
4. Rotate regularly

### Scraping Ethics

**Guidelines**:
1. Respect `robots.txt` (Threads likely disallows)
2. Throttle aggressively (default 2 seconds minimum)
3. Cache results (avoid repeat requests)
4. Public data only (no auth bypass attempts)
5. Personal use only (no commercial scraping)

### Error Messages

**Avoid leaking**:
- Session cookie values
- Internal paths
- Stack traces with sensitive data

**Safe error messages**:
```python
return create_text_result(
    json.dumps({"error": "Profile not found"}, indent=2),
    is_error=True
)
```

## Maintenance Checklist

### Monthly
- [ ] Test with @zuck, @meta and other known profiles
- [ ] Check if selectors still work
- [ ] Verify JSON structure unchanged
- [ ] Update dependencies (cautiously)

### After Threads Updates
- [ ] Inspect `__NEXT_DATA__` JSON structure
- [ ] Update `ThreadsSelectors` if HTML changed
- [ ] Update JSON paths in parsers if needed
- [ ] Test all 6 MCP tools

### Before Release
- [ ] Test all tools with real data
- [ ] Verify throttling working
- [ ] Check cache TTL appropriate
- [ ] Review error messages (no leaks)
- [ ] Update version in `__init__.py` and `pyproject.toml`

## Debugging

### Enable Debug Logging

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Inspect Raw HTML

```python
client = ThreadsPublicClient()
html = client._get("https://threads.net/@little_hakr").text
print(html[:1000])  # First 1000 chars
```

### Inspect __NEXT_DATA__

```python
from bs4 import BeautifulSoup
from threads_scraper_mcp.parsers import _extract_next_data

soup = BeautifulSoup(html, "lxml")
data = _extract_next_data(soup)
import json
print(json.dumps(data, indent=2)[:2000])
```

### Check Cache

```python
from threads_scraper_mcp.cache import _cache
print(f"Cache size: {len(_cache)}")
print(f"Cache keys: {list(_cache.keys())[:5]}")
```

### Test MCP Locally

```bash
# Run server
python run_mcp_server.py

# In another terminal, send MCP request
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | python run_mcp_server.py
```

## Future Enhancements

1. **Pagination**: Implement infinite scroll or API pagination
2. **Async client**: Use `httpx.AsyncClient` throughout
3. **Persistent cache**: SQLite or file-based cache
4. **Retry logic**: Exponential backoff for 429 errors
5. **Bulk operations**: Parallel profile/thread fetching
6. **Thread trees**: Full conversation threading
7. **Media download**: Save images/videos locally
8. **Advanced analytics**: Trend detection, topic modeling

## Contributing Guidelines

1. **Preserve patterns**: Follow existing code style
2. **Test thoroughly**: Manual testing with real URLs required
3. **Document changes**: Update both README.md and CLAUDE.md
4. **Centralize selectors**: Always update `ThreadsSelectors` class
5. **Graceful degradation**: Return partial data, don't fail completely
6. **Error messages**: Clear, user-friendly, no sensitive data

## Dependencies

### Required
- `playwright>=1.40`: Headless browser automation
- `beautifulsoup4>=4.12`: HTML parsing (fallback)
- `lxml>=5.2`: Fast XML/HTML parser backend
- `pydantic>=2.8`: Data validation (v2 only)
- `vaderSentiment>=3.3`: Sentiment analysis
- `python-dateutil>=2.9`: Flexible date parsing
- `cachetools>=5.3`: TTL cache implementation
- `mcp==1.10.0`: MCP protocol (locked version)

### Post-Install
```bash
# Required: Install Chromium browser for Playwright
playwright install chromium
```

### Dev (Optional)
- `pytest>=8.3`: Testing framework
- `pytest-asyncio>=0.23`: Async test support
- `respx>=0.20`: HTTP mocking for tests

## License

MIT License - See LICENSE file

## Contact

For questions or issues, check:
- GitHub Issues
- CLAUDE.md (this file)
- README.md (user docs)

---
> Source: [dkyazzentwatwa/threads-mcp-crawler](https://github.com/dkyazzentwatwa/threads-mcp-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
