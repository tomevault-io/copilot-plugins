## ministryofechoes

> Async/await patterns and performance rules. Require async I/O, prevent blocking, enforce connection pooling and caching.


# Async and Performance Rules

## Core Principle
Async/await for all I/O operations. Don't block the main loop. Cache aggressively.

## Async Rules

### I/O Operations
- **All I/O operations must use async/await**
- HTTP requests: Use httpx.AsyncClient
- File I/O: Use aiofiles
- Database: Use async database libraries
- Never use blocking I/O in async functions

### Async Pattern
```python
#  Correct
async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com")
        return response.json()

# Wrong - blocking
async def fetch_data():
    response = requests.get("https://api.example.com")  # Blocking!
    return response.json()
```

### Parallel Operations
- Use `asyncio.gather()` for parallel operations
- Process multiple outputs concurrently
- Don't wait sequentially when operations are independent

### Connection Pooling
- Reuse HTTP clients (don't create new clients for each request)
- Use connection pooling
- Configure timeouts appropriately

## Performance Rules

### Caching
- **Cache aggressively**: TTS audio, generated images
- Use disk-based caching with metadata
- Check cache before making API calls
- Implement cache cleanup (size limits, TTL)

### API Usage
- Respect rate limits (implement backoff)
- Batch operations when possible
- Monitor API usage and costs
- Cache responses to minimize API calls

### Resource Management
- Clean up resources properly (close files, connections)
- Limit concurrent operations
- Monitor memory usage
- Implement resource limits

### What NOT to Do
- Don't block async functions with synchronous I/O
- Don't create new HTTP clients for each request
- Don't skip caching (API calls are expensive)
- Don't process operations sequentially when they can be parallel

### What TO Do
-  Use async/await for all I/O
-  Reuse HTTP clients
-  Cache API responses
-  Use asyncio.gather() for parallel operations
-  Monitor performance and optimize bottlenecks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/melisay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
