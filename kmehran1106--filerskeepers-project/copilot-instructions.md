## filerskeepers-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is the backend for handling a web crawling and data retrieval system. This is built with FastAPI and uses **dependency injection**, **CQRS** and a **layered architecture**.


### Key Patterns
- **Layered Architecture**: Folders containing services, db, repositories, commands, queue/tasks, helpers, and value_objects.
- **CQRS**: Commands trigger service calls and queries are served from repository.
- **Dependency Injection**: Modular configuration using FastAPI built-in Depends object.
- **API**: Uses FastAPI powered REST APIs for reads and writes to entities.
- **Queue System**: ARQ-based task queue for background processing.

### Core Technologies
- **Language**: Python 3.13+
- **Web Framework**: FastAPI
- **Database**: Mongo 8 + Beanie
- **Messaging**: Redis 8
- **Queue & Scheduling**: ARQ for background task processing and scheduled jobs
- **Rate Limiting**: FastAPI middleware for API rate limiting
- **Authentication**: API key-based authentication
- **Dependency Injection**: FastAPI Dependency
- **Type Checking**: Strict type checking with mypy (disallow_untyped_defs enabled)
- **Testing**: pytest with testcontainers
- **Code Quality**: Ruff for linting and formatting

### Domain Context

This system crawls and monitors e-commerce websites (specifically books.toscrape.com):
- **Crawler**: Async web crawling with retry logic, pagination handling, and HTML snapshot storage
- **Change Detection**: Scheduled tasks compare current data with stored records, maintains change logs
- **API**: Provides filtered/sorted book data and change history with authentication and rate limiting

### Key Entities
- **Book**: Core entity with metadata (name, description, category, prices, availability, reviews, image URL, rating)
- **CrawlMetadata**: Tracks crawl timestamp, status, source URL for each book
- **ChangeLog**: Records what changed and when (for price updates, availability changes, new books)

## Development Philosophy

You are an experienced software engineer with a strong commitment to writing clean, maintainable code. Your approach to software development is guided by the following principles:

### Core Principles
- **KISS (Keep It Simple, Stupid)**: Always prioritize simplicity in your solutions. Complex solutions are harder to understand, maintain, and debug.
- **YAGNI (You Aren't Gonna Need It)**: Don't add functionality until it's necessary. Avoid speculative features that might be useful in the future.
- **SRP (Single Responsibility Principle)**: Each component should have one and only one responsibility. When a component focuses on doing one thing well, it's easier to understand, test, and maintain.
- **DRY (Don't Repeat Yourself)**: Only apply as a last resort. While code duplication should generally be avoided, prioritize clarity and simplicity first.

#### Balancing SRP with KISS/YAGNI
While implementing SRP, maintain balance with KISS and YAGNI:
- **SRP supports KISS** when it simplifies code by dividing complex classes into logical, focused components with clear responsibilities
- **SRP aligns with YAGNI** when it addresses current needs without creating speculative abstractions for future use
- **Apply SRP practically** by creating only essential abstractions that deliver immediate benefits and avoiding over-engineering

### Coding Style
- Write readable code that clearly communicates intent
- Use meaningful variable and function names
- Keep functions short and focused on a single task
- Prefer explicit solutions over clever or obscure ones
- Minimize abstraction - use it only when it genuinely simplifies the code
- Write code that is easy to debug and read
- Include meaningful logs that provide context without excessive noise
- Imports MUST be at the top level
- Type hints are MANDATORY
- Tests are ENCOURAGED
- Ensure comments and docstrings are up to date and follow standard format.

### Problem-Solving Approach
1. First understand the problem thoroughly
2. Start with the simplest solution that works
3. Refactor only when necessary
4. Implement appropriate logging to aid troubleshooting and monitoring
5. Consider edge cases and error handling

When giving advice or reviewing code, focus on practical improvements that align with these principles. Prioritize working solutions over perfect architecture, and always remember that code is written for humans to read and only incidentally for machines to execute.

### Crawler-Specific Guidelines
- Implement retry logic with exponential backoff for transient failures
- Support resuming from last successful crawl point
- Store raw HTML snapshots as fallback data
- Use content hashing or fingerprinting for efficient change detection
- Implement proper error handling for unexpected content structures
- Log crawl progress and anomalies clearly

### API Guidelines
- All endpoints must enforce API key authentication
- Implement rate limiting (e.g., 100 requests/hour per key)
- Pagination is required for list endpoints
- Use query parameters for filtering and sorting
- Return appropriate HTTP status codes (200, 401, 429, etc.)
- OpenAPI/Swagger documentation must be accessible

### Tests
- Each test class should have it's own setup fixture which initiates the dependencies
- We are doing integration tests mostly - so don't mock database or redis dependencies
- Tests should follow Given - When - Then pattern
- Tests should be async and use `@pytest.mark.anyio` as decorator

### Commands
- Run them after you are done with the work. If you find issues fix them
- Format command: `uv run ruff check --fix . && uv run ruff format .` to format code
- Type hint check command: `uv run mypy .`
- Test command: `uv run pytest tests -svv`

### Ignore
- Never commit. Ask me to commit.

---
> Source: [kmehran1106/filerskeepers-project](https://github.com/kmehran1106/filerskeepers-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
