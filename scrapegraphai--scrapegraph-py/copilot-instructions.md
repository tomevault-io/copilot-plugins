## scrapegraph-py

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**scrapegraph-py** is the official Python SDK for the ScrapeGraph AI API. It provides a Python client for intelligent web scraping powered by AI.

## Repository Structure

```
scrapegraph-py/
├── scrapegraph_py/         # Python SDK source
├── tests/                  # Test suite
├── examples/               # Usage examples
├── docs/                   # MkDocs documentation
├── cookbook/               # Tutorials and recipes
└── .github/workflows/      # CI/CD
```

## Tech Stack

- **Language**: Python 3.10+
- **Package Manager**: uv (recommended) or pip
- **Core Dependencies**: requests, pydantic, python-dotenv, aiohttp
- **Testing**: pytest, pytest-asyncio, pytest-mock, aioresponses
- **Code Quality**: ruff
- **Build**: hatchling
- **Release**: semantic-release

## Commands

```bash
# Install
uv sync

# Test
uv run pytest tests/ -v

# Format & lint
uv run ruff format src tests
uv run ruff check src tests --fix

# Build
uv build
```

## Before completing any task

Always run these commands before committing or saying a task is done:

```bash
uv run ruff format src tests
uv run ruff check src tests --fix
uv build
uv run pytest tests/ -v
```

No exceptions.

## Versioning & Releases

Use **semantic-release** to manage versions. Prefer **patch bumps (+0.0.1)** as much as possible — reserve minor (+0.1.0) for genuinely new user-facing features and major (+1.0.0) for breaking changes only.

Commit message conventions that drive the bump:

- `fix:` → patch (+0.0.1) — **default choice** for most changes
- `docs:` → patch (+0.0.1) — documentation updates
- `chore:` → patch (+0.0.1) — maintenance, tooling, deps
- `feat:` → minor (+0.1.0) — only for meaningful new features
- `BREAKING CHANGE:` footer or `!` suffix → major (+1.0.0) — only for breaking API changes

When in doubt, frame the change as `fix:`, `docs:`, or `chore:` to keep the bump at patch level.

## Architecture

**Core Components:**

1. **Clients** (`scrapegraph_py/`):
   - `client.py` - Sync client
   - `async_client.py` - Async client

2. **Models** (`scrapegraph_py/models/`):
   - Pydantic models for request/response validation

3. **Config** (`scrapegraph_py/`):
   - `config.py` - API base URL, timeouts
   - `exceptions.py` - Custom exceptions

## API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| SmartScraper | `smartscraper()` | AI data extraction |
| SearchScraper | `searchscraper()` | Multi-URL search |
| Markdownify | `markdownify()` | HTML to Markdown |
| Crawler | `crawler()` | Sitemap & crawling |
| AgenticScraper | `agentic_scraper()` | Browser automation |
| Scrape | `scrape()` | Basic HTML fetch |
| Credits | `get_credits()` | Balance check |

## Adding New Endpoint

1. Add models in `scrapegraph_py/models/`
2. Add sync method to `client.py`
3. Add async method to `async_client.py`
4. Export in `models/__init__.py`
5. Add tests in `tests/`

## Environment Variables

- `SGAI_API_KEY` - API key for authentication

## Usage

```python
from scrapegraph_py import Client

client = Client(api_key="your-key")
response = client.smartscraper(
    website_url="https://example.com",
    user_prompt="Extract title"
)
print(response.result)
```

## Links

- [API Docs](https://docs.scrapegraphai.com)
- [PyPI](https://pypi.org/project/scrapegraph-py/)

---
> Source: [ScrapeGraphAI/scrapegraph-py](https://github.com/ScrapeGraphAI/scrapegraph-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
