## burnar

> MIT Licensed secure ephemeral content sharing application.

# burnar - Burn After Reading

MIT Licensed secure ephemeral content sharing application.

## Project Overview

**burnar** is a web application add-on that provides secure, ephemeral content sharing with automatic expiration. Content "burns after reading" - self-destructing after being viewed or after a time limit expires.

See `ROADMAP.md` for known issues, current and future plans.

### Key Characteristics

- **Add-on Architecture**: Designed to integrate with existing web installations
- **Target Platform**: Linux with HTTPS (security-first approach)
- **License**: MIT
- **Security Focus**: End-to-end encryption, no persistent storage of sensitive content

## Development Environment
- **Language**: Python 3.11
- **Dependency Management**: `uv`
- **Containerization**: Docker (via `docker-compose`)

## Coding Standards
- **Path Formatting**: ALWAYS use POSIX format (e.g., `./path/to/file`) for file paths. NEVER use Windows backslashes (e.g., `.\path\to\file`), even on Windows.

## Deployment Operations
- **Restarting the Service**: After updating the application code or configuration, you **MUST** restart the service for changes to take effect:
  ```bash
  sudo systemctl restart burnar
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XistGG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
