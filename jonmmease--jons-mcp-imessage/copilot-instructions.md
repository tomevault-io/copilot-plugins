## jons-mcp-imessage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A local MCP server for iMessage

This is a FastMCP server that exposes tools through the Model Context Protocol (MCP), enabling AI assistants to interact with your service.

## Build and Development Commands

```bash
# Install dependencies
uv pip install -e .

# Install with dev dependencies
uv pip install -e ".[dev]"

# Run the server
uv run jons-mcp-imessage

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_example.py

# Run a single test
uv run pytest tests/test_example.py::test_hello_world

# Type check
uv run mypy src/jons_mcp_imessage

# Format code
uv run black src tests

# Lint code
uv run ruff check src tests
```

## Architecture

### Package Structure

```
src/jons_mcp_imessage/
├── __init__.py          # Package exports
├── constants.py         # Configuration constants
├── exceptions.py        # Custom exception classes
├── utils.py             # Utility functions (pagination, file URIs)
├── server.py            # FastMCP server setup, main()
└── tools/
    ├── __init__.py      # Re-exports all tools
    └── example.py       # Example tool implementation
```

### Core Components

- **`server.py`**: FastMCP server setup with tool registration and main entry point

- **`tools/`**: MCP tool functions. Each tool should be an async function with docstrings (used as tool descriptions)

### Key Patterns

- **File URI handling**: `ensure_file_uri()` in `utils.py` converts paths to `file://` URIs

- **Pagination**: `apply_pagination()` provides consistent limit/offset handling for list-returning tools

- **Tool docstrings**: FastMCP uses function docstrings as tool descriptions, so keep them clear and concise

### Adding New Tools

1. Create a new file in `tools/` or add to existing file
2. Write an async function with type hints and a docstring
3. Export from `tools/__init__.py`
4. Register in `server.py` with `mcp.tool(your_function)`

Example:
```python
async def my_tool(param: str) -> str:
    """Brief description of what this tool does.

    Args:
        param: Description of the parameter.

    Returns:
        Description of what's returned.
    """
    return f"Result: {param}"
```

### Contact Name Enrichment

The server automatically enriches message and conversation responses with contact names from the macOS Contacts app. This feature requires **Contacts permission** and provides a better user experience by showing names instead of phone numbers/emails.

#### Contact Name Features

**Message responses** include a `contact_name` field:
```python
{
    "rowid": 12345,
    "text": "Hello!",
    "sender": "+15551234567",
    "contact_name": "John Smith",  # Resolved from Contacts
    ...
}
```

**Conversation responses** include a `participant_names` field:
```python
{
    "chat_id": 42,
    "participants": ["+15551234567", "+15559876543"],
    "participant_names": ["John Smith", "Jane Doe"],  # Resolved names
    ...
}
```

#### Contact Lookup Tool

Use `lookup_contact` to explicitly resolve a phone number or email to a contact name:

```python
# Look up a contact by phone
lookup_contact(phone_or_email="+15551234567")
# Returns: {"name": "John Smith", "matched_handle": "+15551234567"}

# Look up by email
lookup_contact(phone_or_email="john@example.com")
# Returns: {"name": "John Smith", "matched_handle": "john@example.com"}

# Contact not found
lookup_contact(phone_or_email="+19999999999")
# Returns: None
```

#### Contact Search Tool

Use `search_contacts` to search for handles (phone/email) in your iMessage database:

```python
# Search for contacts by phone
search_contacts(query="555-1234")
# Returns matches from your iMessage conversations

# Search by email
search_contacts(query="example.com", limit=10)
```

#### Permission Requirements

Contact name enrichment requires **macOS Contacts permission**:

1. Open **System Settings** → **Privacy & Security** → **Contacts**
2. Add the application running the server (Terminal.app, Claude Desktop.app, etc.)
3. Restart the application

If Contacts permission is not granted:
- Contact names will be `null` in message/conversation responses
- `lookup_contact` will return an error explaining the permission requirement
- All other functionality continues to work normally (graceful degradation)

#### Privacy Notes

- **Contact names come from YOUR Contacts app** - they reflect your personal contact list
- **Names are NOT stored** in the search index or any database
- **Names are resolved at runtime** each time messages are retrieved
- **First load caches all contacts** from all Contacts databases (main + iCloud/CardDAV sources)
- **Cache is in-memory only** and refreshed on server restart

### Search Tools

The `search_messages` tool supports hybrid search combining full-text search (FTS5) with semantic search:

```python
# Basic search
search_messages(query="dinner plans")

# With filters
search_messages(
    query="meeting",
    sender="+15551234567",
    after_date="2024-01-01",
    search_mode="hybrid",
    limit=50
)

# Keyword-only (no API key needed)
search_messages(query="exact phrase", search_mode="keyword")

# Semantic-only (requires OPENAI_API_KEY)
search_messages(query="conceptual topic", search_mode="semantic")
```

Index management tools:

```python
# Check index status
search_index_status()

# Rebuild if needed (background task)
rebuild_search_index(background=True)

# Check rebuild progress
get_rebuild_progress()

# Check backfill progress (initial background indexing)
get_backfill_progress()
```

The search system supports:
- **FTS5 query syntax**: Phrases (`"exact"`, `word*`, `NEAR`, `AND`/`OR`/`NOT`)
- **Hybrid mode**: Combines keyword and semantic results using Reciprocal Rank Fusion (RRF)
- **Date filters**: ISO8601 format dates for `after_date` and `before_date`
- **Participant filtering**: Filter group messages by chat participants

---
> Source: [jonmmease/jons-mcp-imessage](https://github.com/jonmmease/jons-mcp-imessage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
