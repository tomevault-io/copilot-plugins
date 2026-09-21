## seoproga

> Python SEO automation project clustering keywords via SERP similarity.

# AGENTS.md — Agent Coding Guidelines for seo-auto-cluster

Python SEO automation project clustering keywords via SERP similarity.
Integrates: **XMLRiver**, **Yandex Webmaster**, **Miratext**, **OpenAI**

## Global Rules

1. **ARCH.md must be read at the start of every new session** — before making any changes, read `ARCH.md` to understand the full project architecture, module purposes, data flow, and API endpoints.

2. **ARCH.md must be updated when developing new features or modifying existing ones** — any addition, change, or removal of modules, services, API endpoints, scripts, database tables, or frontend pages must be documented in `ARCH.md` within the same commit. Keep descriptions accurate and current.

---

## Build, Test & Lint Commands

```bash
# Setup
.venv\Scripts\activate && pip install -r requirements.txt

# Run (4 modes: 1=XMLRiver, 2=Yandex, 3=Miratext, 4=Full workflow)
echo 1 | python main.py

# Streamlit dashboard
streamlit run streamlit_app.py

# Testing
pytest                    # All tests
pytest tests/test_x.py    # Single file
pytest tests/x.py::test_func -v  # Single test
pytest -k "keyword"       # By pattern

# Lint & format
pip install ruff mypy
ruff check . && ruff check --fix . && ruff format .
mypy services/ config.py
```

---

## Code Style Guidelines

- **PEP 8** + Python 3.10+ idioms; use `X | None` over `Optional[X]`
- **Type hints** required on all public functions
- **Docstrings** on public classes/methods (Google style)
- **No comments** unless explicitly requested
- **Max 50 lines** per function; extract helpers when needed

### Naming Conventions

| Element        | Convention        | Example                      |
|----------------|-------------------|------------------------------|
| Classes        | PascalCase        | `XmlriverClient`            |
| Functions/vars | snake_case        | `fetch_serp()`              |
| Constants      | SCREAMING_SNAKE   | `SERP_TOP_N`               |
| Private        | _leading_underscore | `_get_conn()`             |
| Modules        | snake_case        | `yandex_webmaster.py`       |

### Imports Order

```python
# 1. Standard lib (alphabetical)
import json
from datetime import datetime
from typing import List, Dict

# 2. Third-party
import requests

# 3. Local app
from config import Config
from services.cache import SERPCache
```

### Error Handling

Catch specific exceptions, log with context, then re-raise. Never swallow silently.

```python
try:
    response = requests.get(url, timeout=60)
    response.raise_for_status()
except requests.exceptions.HTTPError as e:
    print(f"HTTP error {e.response.status_code}: {e}")
    raise
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
    raise
```

### Database Patterns

Always use context managers and parameterized queries:

```python
with sqlite3.connect(self.db_path) as conn:
    cursor = conn.execute(
        "SELECT urls FROM serp_cache WHERE cache_key = ?",
        (key,)
    )
```

### Async Patterns

Use `aiohttp` for async HTTP; always use async context managers:

```python
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()
```

---

## File Structure

```
seo-auto-cluster/
├── main.py           # CLI entry (4 modes)
├── config.py         # Config + validation
├── streamlit_app.py  # Dashboard
├── services/
│   ├── clustering.py      # SERP similarity + Jaccard
│   ├── cache.py           # MySQL SERP cache (lazy conn)
│   ├── xmlriver_client.py # XMLRiver API (requests.Session, lazy init)
│   ├── yandex_webmaster.py# Yandex WM API (HTTP timeout 30s)
│   ├── semantic_core.py   # PostgreSQL storage
│   ├── miratext_client.py # Miratext API
│   ├── seo_agent.py       # OpenAI LLM agent
│   ├── seo_workflow.py    # Full pipeline
│   └── page_content_manager.py
├── utils/
│   ├── bootstrap.py   # Script entry: chdir, sys.path, stdout fix
│   └── helpers.py     # extract_domain, clean_url, safe_divide, safe_print
├── scripts/  # 27 scripts, all use bootstrap()
└── tests/    # 10 test modules (incl. xmlriver, yandex, miratext)
```

---

## Key Patterns & Config

| Pattern          | Value                                      |
|------------------|--------------------------------------------|
| SERP similarity  | Jaccard + position-weighted (0.7/0.3)     |
| Cache TTL        | 7 days (`CACHE_TTL_DAYS`)                 |
| Cluster threshold| 0.4 (`SIMILARITY_THRESHOLD`)              |
| SERP Top N       | 10 URLs (`SERP_TOP_N`)                    |
| LLM Model        | `gpt-4o-mini`, temp 0.2, 8192 tokens      |
| XMLRiver region  | 213 (Moscow)                              |

### Required ENV Variables

| Mode         | Variables                                      |
|--------------|------------------------------------------------|
| XMLRiver     | `XMLRIVER_USER`, `XMLRIVER_KEY`                |
| Yandex       | `YANDEX_OAUTH_TOKEN`, `YANDEX_SITE_URL`        |
| Miratext     | `MIRATEXT_API_KEY`, `OPENAI_API_KEY`           |
| Full         | Above + `PG_PASSWORD`                         |

---

## Common Pitfalls

1. Debug prints — remove before committing
2. Hardcoded values — use `Config` class constants
3. Missing type hints on public functions
4. Resource leaks — use `with` for all DB connections
5. Blocking sync calls — use `aiohttp` for async
6. URL encoding — XMLRiver needs `&` → `%26`
7. Minus-words bulk update — always use `IN (...)` not loop per keyword
8. XmlriverClient uses lazy session — don't access `.session` before `__init__`

---

## Git

**Ignore:** `.env`, `data/`, `results/`, `__pycache__/`, `.venv/`, `node_modules/`, `.ruff_cache/`, `.mypy_cache/`

**Commit format:** `type: description` (feat/fix/refactor/test/docs)

---
> Source: [Barnick84/seoproga](https://github.com/Barnick84/seoproga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
