## search-api-webui

> **search-api-webui** is a full-stack web application for testing and comparing multiple Search API providers. It can be packaged as a native macOS/Windows app, Android APK, or installed via pip.

# Project Overview

**search-api-webui** is a full-stack web application for testing and comparing multiple Search API providers. It can be packaged as a native macOS/Windows app, Android APK, or installed via pip.

---

# Project Structure

```
search-api-webui/
├── search_api_webui/        # Python backend package
│   ├── app.py               # Flask app — all API routes, CLI entry point (main())
│   ├── providers.yaml       # YAML config for all built-in search providers
│   ├── ruff.toml            # Ruff linter/formatter config (line-length: 120)
│   └── providers/
│       ├── __init__.py      # load_providers() factory
│       ├── base.py          # BaseProvider ABC + helper utilities
│       └── generic.py       # GenericProvider — single concrete provider implementation
├── frontend/                # React + Vite frontend
│   ├── src/
│   │   ├── App.jsx          # Router: /, /config, /arena
│   │   ├── SearchPage.jsx   # Main search page with SSE streaming
│   │   ├── ConfigPage.jsx   # API key + advanced settings
│   │   ├── ArenaPage.jsx    # Side-by-side provider comparison
│   │   ├── components/      # Button, Input, Card, Badge, ResultItem
│   │   ├── lib/utils.js     # cn() utility: clsx + tailwind-merge
│   │   └── utils/engineHistory.js  # localStorage engine selection history
│   ├── vite.config.js       # Dev proxy /api -> http://localhost:8889
│   └── package.json
├── main.py                  # Android entry point
├── pyproject.toml           # Package metadata, deps, entry point
├── Makefile                 # Build targets: dev, dmg, exe, apk, build-wheel
├── buildozer.spec           # Android APK build config
└── SearchAPIWebUI.spec      # PyInstaller spec for macOS/Windows
```

---

# Key Architectural Patterns

## Backend

- **YAML-driven providers**: All providers in `providers.yaml`; `GenericProvider` handles everything via template substitution (`{query}`, `{api_key}`, `{limit}`) and JMESPath response mapping. No provider-specific Python code needed.
- **SSE streaming**: `/api/search` runs HTTP calls in a background thread, streams `warming_up` / `searching` status events before the final `result` event.
- **Connection pre-warming**: `GenericProvider._ensure_connection()` sends a HEAD request to pre-establish HTTPS connections (can be disabled per-provider via `skip_warmup`).
- **User data**: Stored in `~/.search-api-webui/config.json` (API keys + settings) and `~/.search-api-webui/search_history.json` (max 1000 entries).

## Frontend

- **React 18** + React Router v6, **Vite 7**, **Tailwind CSS v3**
- In production, Flask serves the pre-built React `dist/` as static files.
- In development, Vite dev server (port 5173) proxies `/api` to Flask (port 8889).

## API Routes (Flask)

| Route | Method | Purpose |
|---|---|---|
| `/api/providers` | GET | All providers with `has_key`, `user_settings`, `details` |
| `/api/search` | POST | Execute search; `stream: true` enables SSE |
| `/api/config` | POST | Save API key + advanced settings for a provider |
| `/api/search-history` | GET | Recent search history (`?prefix=` filter supported) |
| `/api/search-history` | POST | Add query or clear history (`query: null`) |
| `/api/browser-open` | POST | Android only: open URL in external browser |

## Provider Search Response Contract

```python
{
    'results': [{'title', 'url', 'snippet', 'site_name', 'site_icon', 'page_age'}],
    'metrics': {'latency_ms', 'server_latency_ms', 'size_bytes'},
    'error': '...'  # optional
}
```

---

# Project Coding Rules

## Python (Backend)

### Indentation
- Use **4 spaces** for indentation (no tabs)
- Configure editor: `indent_size = 4`

### Line Spacing
- Two blank lines between top-level class/function definitions
- One blank line between method definitions within a class
- One blank line between logical code blocks
- Maximum line length: 100 characters

### Comments
- **All comments must be in English**
- Use docstrings for all public classes, methods, and functions
- Inline comments for complex logic (explain "why", not "what")
- Example:
  ```python
  def process_data(self, data):
      """
      Process input data and return normalized result.

      Args:
          data: Raw input data dictionary

      Returns:
          Normalized data dict with default values applied
      """
      # Skip validation for cached data to improve performance
      if data.get('cached'):
          return data
      ...
  ```

### Naming Conventions
- **Classes**: `PascalCase` (e.g., `GenericProvider`)
- **Functions/Variables**: `snake_case` (e.g., `search_results`, `ensure_connection`)
- **Private Members**: Leading underscore (e.g., `_connection_ready`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `DEFAULT_TIMEOUT`)

### Imports
- Standard library imports first, then third-party, then local
- Group imports with blank lines between groups
- Use absolute imports

### Error Handling
- Use specific exception types when possible
- Log errors with descriptive messages
- Return consistent error response format

---

## JavaScript/JSX (Frontend)

### Indentation
- Use **2 spaces** for indentation

### Comments
- **All comments must be in English**
- Use JSDoc for component props documentation

### Naming
- **Components**: `PascalCase` (e.g., `SearchBox`)
- **Variables/Functions**: `camelCase` (e.g., `handleSearch`, `searchResults`)
- **Constants**: `UPPER_SNAKE_CASE` or `camelCase` based on context

---

## YAML Configuration

### Indentation
- Use **2 spaces** for indentation

### Structure
- Clear hierarchical structure
- Use lowercase for all keys
- Descriptive key names

---

## General Rules

### Code Style
- Use **single quotes** `'` for all Python strings and doc strings
- Remove trailing whitespace
- Add newline at end of file
- Keep files focused (single responsibility)

### Version Control
- Meaningful commit messages
- Review before commit

### Testing
- Add tests for new functionality
- Run existing tests before pushing

---

## Tooling

### Virtual Environment

The Makefile uses `venv-dev` as the designated Python virtual environment (defined as `VENV := venv-dev`).
**All Python package and tool installations, as well as all tool executions (pre-commit, ruff, python, etc.), must be performed inside `venv-dev`**, not the system Python or any other environment.

```bash
# Create venv-dev and install all dependencies (done automatically by make dev / make build-wheel)
make dev

# Install a package manually
venv-dev/bin/pip install <package>
```

### Development

```bash
make dev          # Start Flask (port 8889) + Vite dev server (port 5173) concurrently
```

### Pre-commit Hooks
```bash
venv-dev/bin/pre-commit run --all-files    # Run all checks manually
venv-dev/bin/pre-commit install            # Install git hooks
```

### Python Linting & Formatting (Ruff)
```bash
venv-dev/bin/ruff format search_api_webui/
venv-dev/bin/ruff check search_api_webui/ --fix
```

### JavaScript/JSX Linting & Formatting
```bash
cd frontend && npx eslint . --fix
cd frontend && npm run format
```

### Building

```bash
make build-wheel   # Build pip-installable wheel
make dmg           # Build macOS .app + DMG (requires PyInstaller + create-dmg)
make exe           # Build Windows installer (requires PyInstaller + Inno Setup)
make apk-debug     # Build Android APK (requires Buildozer)
```

---
> Source: [querit-ai/search-api-webui](https://github.com/querit-ai/search-api-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
