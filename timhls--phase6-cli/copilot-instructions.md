## phase6-cli

> A modern Python CLI for managing vocabulary in the Phase-6 web application. This tool reverse-engineers the private Phase-6 SPA API, combining browser automation (Playwright) for authentication with direct REST API calls for fast vocabulary CRUD operations.

# pyphase6 - Copilot Instructions

A modern Python CLI for managing vocabulary in the Phase-6 web application. This tool reverse-engineers the private Phase-6 SPA API, combining browser automation (Playwright) for authentication with direct REST API calls for fast vocabulary CRUD operations.

## Build, Test, and Lint

**This project uses `uv` exclusively. Never use `pip` or `poetry`.**

```bash
# Install dependencies
uv sync

# Install Playwright browsers (required for authentication)
uv run playwright install chromium

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_client.py

# Run a single test function
uv run pytest tests/test_client.py::test_get_api_headers_success

# Format code
uv run ruff format .

# Lint code (with auto-fix)
uv run ruff check . --fix

# Type check
uv run mypy .
```

**Code Quality Expectation**: Don't act as a linter. Always run the automated tools above to verify changes.

## Architecture

### Three-Layer Structure

1. **CLI Layer** (`cli.py`): Typer commands that provide the user interface
   - Handles user input/output with Rich terminal UI (tables, progress bars)
   - Validates authentication state before delegating to client
   - Pattern: Commands call `get_authenticated_client()` before any API operations

2. **Client Layer** (`client.py`): The `Phase6Client` class that manages API communication
   - **Authentication**: Uses Playwright browser automation to log in and extract session tokens from localStorage
   - **Session Management**: Stores session state in `~/.config/pyphase6/session.json`
   - **API Communication**: Uses Playwright's request context (NOT httpx) for all API calls after authentication
   - Pattern: Each API method uses `_get_api_headers()` to extract tokens from saved session file

3. **Model Layer** (`models.py`): Pydantic models for data validation
   - All API request/response data is validated using Pydantic models
   - Models mirror the Phase-6 API structure (e.g., `Subject`, `VocabItem`, `CardContent`)

### Key Architectural Decisions

**Why Playwright for API calls?**
- The authentication flow extracts session tokens from browser localStorage (not cookies)
- Playwright's request context is used for ALL API calls (not just login) to ensure consistency
- Each API method creates a fresh `sync_playwright()` context with the extracted headers
- Pattern: `with sync_playwright() as p: ctx = p.request.new_context(...)`

**Session Management Flow:**
1. `login()` → Opens headless browser → Saves session with `context.storage_state()`
2. `_get_api_headers()` → Reads saved session → Extracts `jossoSessionId`, `userDnsId`, `email` from localStorage JSON
3. All API methods → Use extracted headers (`x-jauthtoken`, `x-lbtoken`, `x-clientid`) for requests

**Error Handling:**
- Custom exceptions: `AuthError` (authentication issues), `APIConnectionError` (API communication issues)
- Pattern: Raise these exceptions from client layer, catch and display with Rich formatting in CLI layer

## Key Conventions

### Testing Patterns

Tests use mocking for browser automation and API responses:
- `@patch` decorators to mock Playwright and file I/O
- Fixtures for common test data (e.g., `mock_session_data`)
- Pattern: Mock the session file reads, not the actual API calls
- All API responses should validate against Pydantic models in tests

### Session File Structure

The session file (`~/.config/pyphase6/session.json`) contains:
- Playwright's storage state (cookies + localStorage)
- Critical data lives in `localStorage["persist:user"]` (nested JSON strings)
- Must be double-parsed: `json.loads(user_state_str)` then `json.loads(user_state["user"])`

### API Request Pattern

Every API method follows this pattern:
```python
def api_method(self, ...):
    headers, owner_id = self._get_api_headers()
    
    with sync_playwright() as p:
        ctx = p.request.new_context(base_url=self.BASE_URL, extra_http_headers=headers)
        resp = ctx.post("/server.integration/endpoint", data={...})
        
        if not resp.ok:
            raise APIConnectionError(...)
        
        data = resp.json()
        if data.get("httpCode") != 200:
            raise APIConnectionError(...)
    
    # Parse response using Pydantic models
    return Model(**data["replyContent"])
```

### CLI Command Pattern

Every CLI command follows this pattern:
```python
@app.command()
def command_name(...):
    """Clear docstring."""
    client = get_authenticated_client()  # Validates session exists
    with console.status("Loading..."):
        try:
            result = client.api_method(...)
            # Display result with Rich (Table, print, etc.)
        except APIConnectionError as e:
            console.print(f"[red]{e}[/red]")
            raise typer.Exit(1)
```

### Import/Export Data Formats

When adding bulk import/export features:
- CSV format: `question,answer` columns (minimal)
- JSON format: Array of objects with `question` and `answer` keys
- Both formats are demonstrated in README examples

## Release Process

This project uses `semantic-release` for automated versioning and GitHub Releases:
- Commits must follow conventional commit format (e.g., `feat:`, `fix:`, `chore:`)
- Version is auto-bumped in `pyproject.toml` on merge to `main`
- Built with `uv build` (produces wheel/sdist in `dist/`)
- Artifacts uploaded to GitHub Releases automatically

## Entry Points

The main CLI entry point is defined in `pyproject.toml`:
```toml
[project.scripts]
pyphase6 = "pyphase6.cli:app"
```

Run commands with `uv run pyphase6 <command>` during development.

---
> Source: [timhls/phase6-cli](https://github.com/timhls/phase6-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
