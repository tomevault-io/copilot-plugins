## titanurl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TitanUrl** is a URL shortening service built with .NET 8, using distributed ID generation and Base62 encoding to create short, unique identifiers for URLs.

### Architecture

The solution consists of three main projects:

1. **TitanUrl.Core** - Core library containing distributed ID generation logic
   - `SnowflakeIdGenerator`: Generates unique 64-bit IDs using Twitter's Snowflake algorithm with configurable machine ID and custom epoch
   - `Base62Converter`: Encodes/decodes long IDs to/from short Base62 strings (0-9, a-z, A-Z = 62 characters)

2. **TitanUrl.Api** - ASP.NET Core web API that will expose URL shortening endpoints
   - Currently scaffolded with dependency injection for PostgreSQL (Npgsql), Redis (StackExchange.Redis), and Dapper ORM
   - Swagger/OpenAPI configured for documentation

3. **TitanUrl.Core.Tests** - xUnit test suite for Core library
   - Tests Snowflake ID generation: uniqueness (single/multi-threaded), monotonic ordering, bit structure
   - Tests Base62 encoding/decoding: individual conversions and round-trip integrity

## Development Commands

### Build
```bash
dotnet build
```

### Test
```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test /p:CollectCoverage=true

# Run specific test file
dotnet test tests/TitanUrl.Core.Tests/TitanUrl.Core.Tests.csproj

# Run single test
dotnet test --filter "FullyQualifiedName~SnowflakeTests.Generate_ShouldReturnUniqueIds_SingleThread"
```

### Run API
```bash
dotnet run --project src/TitanUrl.Api/TitanUrl.Api.csproj
```

### Clean
```bash
dotnet clean
```

## Key Implementation Details

### Snowflake ID Structure
- 64-bit distributed ID composed of: timestamp (41 bits) | machine ID (10 bits) | sequence (12 bits)
- Thread-safe via lock on `NextId()` method
- Handles clock skew and sequence overflow (waits for next millisecond if sequence rolls over)
- Requires custom epoch initialization (e.g., system launch date)

### Base62 Encoding
- Uses alphabet: "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" (62 chars)
- Bidirectional: encode (long → string) and decode (string → long)
- Can represent 64-bit integers in ~11 characters (much shorter than decimal)

### Testing Strategy
- xUnit framework with Theory and Fact test types
- Snowflake tests validate: uniqueness under concurrency (100K+ IDs), monotonic ordering, bit extraction
- Base62 tests use InlineData for parametrized inputs, plus round-trip integrity verification

## Dependencies

**TitanUrl.Api**:
- `Dapper` (2.1.66) - Lightweight ORM for SQL queries
- `Npgsql` (10.0.1) - PostgreSQL data provider
- `StackExchange.Redis` (2.10.14) - Redis client for caching
- `Microsoft.AspNetCore.OpenApi` - OpenAPI support
- `Swashbuckle.AspNetCore` (6.6.2) - Swagger documentation

**TitanUrl.Core.Tests**:
- xUnit (2.5.3) - Testing framework
- Microsoft.NET.Test.Sdk (17.8.0) - Test runner
- coverlet.collector (6.0.0) - Code coverage

## API Endpoints

### Write Path (Creating Short URLs)
**POST /api/shorten**
- Request: `{ "url": "https://example.com/very/long/path" }`
- Response: `{ "shortCode": "k9J3z", "shortUrl": "http://localhost:5000/k9J3z" }`
- Logic: Snowflake ID → Base62 Encode → DB Insert + Redis Cache (no collision checks)

### Read Path (Following Short URLs)
**GET /api/{shortCode}**
- Redirects to original URL with HTTP 302
- Two-tier lookup: Redis Cache (fast) → PostgreSQL (slow) → Read Repair
- Fire-and-forget analytics: writes click event to Redis Stream
- Aggregated analytics processed by background worker

## Database Tables

**url_mappings**: Stores URL shortening mappings
- `id` (BIGINT): Snowflake ID (primary key)
- `original_url` (TEXT): Full URL
- `short_code` (VARCHAR): Base62 encoded short code
- `created_at` (TIMESTAMP): Creation timestamp
- Index on `short_code` for fast lookups

**url_analytics**: Stores click statistics
- `url_id` (BIGINT): Snowflake ID (primary key, references url_mappings)
- `click_count` (BIGINT): Total clicks
- `last_updated` (TIMESTAMP): Last update time

## Background Services

**AnalyticsWorker**: Processes click events in batch
- Reads events from Redis Stream `stream:clicks`
- Aggregates clicks per URL in-memory (100 events at a time)
- Batch updates PostgreSQL with upsert logic (single transaction)
- Acknowledges processed events and logs batch size

## Architecture Patterns

**Write Path (Shorten)**:
1. Generate ID (Snowflake - 0.0001ms)
2. Encode to Base62 (0.0001ms)
3. Insert to DB + Cache (async, non-blocking)
4. Return short URL

**Read Path (Redirect)**:
1. Check Redis (cache layer, ~1ms)
2. Fall back to DB if miss (read repair)
3. Fire-and-forget analytics event to Redis Stream
4. Return 302 redirect immediately

**Analytics (Background)**:
- Reads 100 events from Redis Stream per cycle
- Aggregates in-memory (e.g., 10 clicks = 1 update of +10)
- Single batch SQL INSERT/UPDATE per transaction
- Handles backoff on error

This design solves the "Justin Bieber Problem": separates user-facing requests from analytics I/O, enabling massive scale.

## Git Workflow

- Main branch: `main`
- Recent commits: Infrastructure setup, write path, read path implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nawedx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nawedx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
