## flutter-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Flutter/Dart documentation MCP (Model Context Protocol) server project designed to provide AI assistants with seamless access to Flutter and Dart documentation. Following Context7's proven approach, the project uses on-demand web scraping with Redis caching to ensure users always get the most current documentation while maintaining fast response times.

## Key Architecture Components

1. **MCP Server**: FastMCP server with on-demand documentation fetching
2. **Redis Caching**: Fast in-memory cache for processed documentation
3. **Web Scraping**: Respectful fetching from api.flutter.dev and api.dart.dev
4. **Pub.dev API**: Official API for package documentation
5. **Processing Pipeline**: Parse → Enrich → Clean → Cache (Context7-style)
6. **Rate Limiting**: 2 requests/second to respect server resources

## Development Commands

```bash
# Project setup (using uv package manager)
uv init mcp-server-flutter-docs
cd mcp-server-flutter-docs
uv add "mcp[cli]" httpx redis beautifulsoup4 structlog

# Start Redis (required for caching)
redis-server  # In a separate terminal

# Development server with MCP Inspector
mcp dev server.py

# Run the server
uv run server.py

# Alternative with traditional Python
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install "mcp[cli]" httpx redis beautifulsoup4 structlog
python server.py
```

## Core Implementation Guidelines

1. **On-Demand Fetching**: Fetch documentation only when requested, like Context7
2. **Redis Caching**: Cache processed docs with appropriate TTLs (24h for APIs, 12h for packages)
3. **Smart URL Resolution**: Pattern matching to resolve queries to documentation URLs
4. **Rate Limiting**: RateLimiter class ensuring 2 requests/second max
5. **Error Handling**: Graceful fallbacks when documentation isn't found
6. **User Agent**: Always identify as "Flutter-MCP-Docs/1.0" with GitHub URL

## Transport and Distribution

- **Primary Transport**: STDIO for local Claude Desktop integration
- **Configuration Path**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Distribution Size**: <10MB lightweight package
- **Dependencies**: Redis required (local or external service)
- **Distribution Methods**: PyPI package, npm package, Docker image
- **Versioning**: Semantic versioning (MAJOR.MINOR.PATCH)

## Implementation Timeline

1. **MVP (4 hours)**: Basic server with Flutter API docs
2. **Week 1**: Add pub.dev support, search functionality
3. **Week 2**: Polish, documentation, and launch
4. **Future**: Stack Overflow, cookbook, version-specific docs

---
> Source: [adamsmaka/flutter-mcp](https://github.com/adamsmaka/flutter-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
