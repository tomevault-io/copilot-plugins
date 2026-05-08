## logging

> Logging standards for Open Edison

## Logging Configuration

Open Edison uses `loguru` for simple, structured logging.

### Usage Pattern

```python
from loguru import logger as log

# Use throughout the codebase
log.info("Starting MCP server: {}", server_name)
log.error("Failed to start server: {}", error)
log.debug("Configuration loaded: {}", config.server.host)
```

### Log Levels

Configure via `config.json`:

```json
{
  "logging": {
    "level": "INFO"
  }
}
```

Available levels: DEBUG, INFO, WARNING, ERROR, CRITICAL

### Best Practices

- Use structured logging with `{}` placeholders
- Log important state changes (server start/stop)
- Include context in error messages
- Use appropriate log levels
- No custom logging configuration - keep it simple

### Examples

```python
# Good
log.info("🚀 Open Edison starting on {}:{}", host, port)
log.error("❌ MCP server {} failed to start: {}", name, error)

# Avoid
print("Starting server...")  # Use logging instead
log.info(f"Server {name}")   # Use {} placeholders
```

---
> Source: [Edison-Watch/open-edison](https://github.com/Edison-Watch/open-edison) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
