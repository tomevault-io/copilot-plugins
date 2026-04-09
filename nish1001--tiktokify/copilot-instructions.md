## tiktokify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TikTokify is a Python CLI that transforms websites/blogs into TikTok-style swipeable HTML viewers. It crawls content, builds a recommendation graph using hybrid similarity algorithms, optionally enriches with LLM summaries and external sources, then generates a single self-contained HTML file.

## Commands

```bash
# Install dependencies (uses uv package manager)
uv sync

# Run the CLI
uv run tiktokify -u <base_url> -o <output.html>

# Full example with enrichment
uv run tiktokify -u https://example.com/blog -o output.html \
  --model gpt-4o-mini \
  --sources hackernews,links \
  --max-depth 2 \
  --verbose

# Run tests
uv run pytest

# Run a single test
uv run pytest tests/test_file.py::test_function -v
```

## Architecture

### Pipeline Flow
```
Spider Crawl → Metadata Extraction → Recommendation Engine → LLM Enrichment → HTML Generation
```

### Module Structure

- **`cli.py`** - Click CLI entry point, orchestrates the full pipeline async
- **`crawler/blog_crawler.py`** - `SpiderCrawler` class for async recursive web crawling with `crawl4ai`, includes metadata extraction (title, date, tags, categories, header images) with multi-platform fallbacks
- **`models/post.py`** - Pydantic models: `Post`, `PostMetadata`, `RecommendationGraph`, `ExternalContentItem`
- **`recommender/`** - Hybrid recommendation engine:
  - `tfidf.py` - TF-IDF vectorization with cosine similarity
  - `metadata.py` - Jaccard similarity on tags/categories
  - `engine.py` - Combines both with configurable weights (default 0.6/0.4)
- **`enrichment/`** - Optional LLM and external content:
  - `llm_enricher.py` - Uses `litellm` for key point extraction + Wikipedia suggestions
  - `providers/` - Pluggable content providers (Wikipedia, HackerNews, link extraction)
  - `base.py` - Abstract `ContentProvider` base class for new sources
- **`generator/html_generator.py`** - Jinja2 template rendering to single HTML file with embedded JSON graph

### Key Patterns

1. **Async-first**: All I/O uses `asyncio` with `asyncio.Semaphore` for rate limiting
2. **Provider pattern**: `ContentProvider` abstract base for extensible external sources
3. **Hybrid similarity**: Weighted combination of content (TF-IDF) and metadata (Jaccard) scores
4. **Multi-platform metadata extraction**: Regex patterns with fallbacks for Jekyll, WordPress, generic HTML

## Tech Stack

- Python 3.11+, Click CLI, Pydantic models, Jinja2 templates
- `crawl4ai` for web scraping, `scikit-learn` for TF-IDF, `httpx` for async HTTP
- `litellm` for LLM abstraction (supports OpenAI, Anthropic, Ollama)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NISH1001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NISH1001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
