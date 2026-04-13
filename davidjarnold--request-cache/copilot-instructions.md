## request-cache

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

```bash
# Build the library
cargo build

# Run all tests
cargo test

# Run a specific test
cargo test test_cached_request

# Generate documentation
cargo doc --open

# Check code without building
cargo check

# Format code
cargo fmt

# Run clippy for linting
cargo clippy
```

## Library Architecture

This is a Rust library that provides HTTP request caching using SQLite as the persistence layer. The library wraps `reqwest` to add automatic caching capabilities with comprehensive error handling, performance optimizations, and a modern API design.

### Core Components

**Main APIs:**
- `cached_request()` - Standalone function that manages its own database connection (returns `Result<Record, RequestCacheError>`)
- `request()` - Uses an explicit database connection for more control (returns `Result<Record, RequestCacheError>`)
- `create_connection()` - Creates and initializes a SQLite database connection (returns `Result<Client, RequestCacheError>`)
- `RequestBuilder` - Builder pattern for configuring requests with method chaining

**Data Structures:**
- `Record` - Represents a cached HTTP response with metadata (uses `Url` and `HttpMethod` types for type safety)
- `RequestCacheError` - Comprehensive error type covering database, HTTP, validation, and parsing errors
- `HttpMethod` - Enum for supported HTTP methods (GET, POST, PUT, DELETE, HEAD, PATCH)

**Database Layer:**
- Uses `async-sqlite` for asynchronous SQLite operations
- Optimized schema with proper indexes: `idx_request_method_expires` for fast lookups
- Automatic table and index creation on connection
- Cache expiration based on Unix timestamps with proper error handling

### Caching Behavior

- Responses are cached by URL and HTTP method combination
- Cache entries have configurable expiration times with validation (must be positive)
- `force_refresh` parameter bypasses cache and makes fresh requests
- Old cache entries for the same URL/method are automatically replaced
- Cache hits are marked with `cached: Some(true)`, misses with `cached: Some(false)`
- HTTP client reuse across requests for better performance
- Request timeouts capped at 5 minutes for safety

### Error Handling & Security

- **No `.unwrap()` calls** - all errors are properly handled and propagated
- **Input validation** - URLs, timeouts, HTTP methods, and database paths are validated
- **Request timeouts** - configurable with sensible defaults and upper limits
- **Type safety** - uses proper types instead of strings for URLs and HTTP methods
- **Comprehensive error types** - detailed error information for debugging

### API Usage Examples

**Builder Pattern (Recommended):**
```rust
let record = RequestBuilder::new()
    .url("https://api.example.com/data")?
    .method(HttpMethod::GET)
    .timeout(300)?
    .user_agent("my-app/1.0")
    .database_path("cache.db")
    .send()
    .await?;
```

**Traditional API:**
```rust
let record = cached_request(
    "https://api.example.com/data".to_string(),
    "GET".to_string(),
    300,
    Some(false),
    Some("my-app/1.0".to_string()),
    Some("cache.db".to_string())
).await?;
```

### Testing Structure

Tests use Tokio runtime and include:
- `TestCleanup` struct with Drop implementation for automatic database file cleanup
- Comprehensive error handling tests for all validation scenarios
- Builder pattern functionality tests
- Performance and caching behavior verification
- Timeout tests use appropriate delays to verify cache expiration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidJArnold)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DavidJArnold)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
