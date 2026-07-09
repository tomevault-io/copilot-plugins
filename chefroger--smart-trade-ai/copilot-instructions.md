## smart-trade-ai

> Foreign Trade Assistant — a B2B Q&A application for trade/manufacturing sales teams. A FastAPI server wrapping **Hermes Agent** with multi-company document libraries, customers, chat memory, skill routing, and a single-page chat UI. Also ships as a desktop app (tradewin) via PyWebView.

# Repository Guidelines

Foreign Trade Assistant — a B2B Q&A application for trade/manufacturing sales teams. A FastAPI server wrapping **Hermes Agent** with multi-company document libraries, customers, chat memory, skill routing, and a single-page chat UI. Also ships as a desktop app (tradewin) via PyWebView.

## Project Structure & Module Organization

```
static/trade_chat.html        Vanilla JS SPA (~3300 lines), served at /trade
server.py / tradewin.py       Entry points (web server / desktop app)
trade/                        Core Python package
  ├── api/                    FastAPI routers (companies, customers, orders, chat, cron, …)
  ├── database.py             SQLite schema + connection
  ├── customer.py             Customer CRUD, dedup, completeness scoring, briefing, health audit
  ├── chat_memory.py          Conversation log + rating + Hindsight bridge
  ├── skill_registry.py       20 skill definitions (triggers, aliases, augment prompts)
  ├── prompt.py               System prompts (full, minimal, OSINT, brand safety)
  ├── prompts.py              Prompt resolution chain (file → DB → code fallback)
  ├── helpers.py              Query builder with skill routing + brand safety injection
  ├── license.py              Ed25519 license validation + self-recovery
  └── osint/                  6-layer B2B due-diligence pipeline
skills/                       Skill markdown files (version controlled)
tests/                        8 test files, ~245 tests
```

## Build, Test, and Development Commands

```bash
# Install (editable) + skills
pip install -e ".[dev]"
install-trade-skills

# Start server
python server.py                          # http://127.0.0.1:9119/trade
python server.py --no-gateway             # skip Hermes Gateway auto-launch
tradewin                                  # desktop app (requires pip install -e ".[desktop]")

# CLI tools
trade-skills-update                       # fetch latest SKILL.md from GitHub
trade-update                              # git pull + pip install + skills + db
trade-backup                              # backup ~/.trade/ to tar.gz

# Testing
python -m pytest tests/ -v                # all tests (asyncio_mode=auto)
python -m pytest tests/test_customer_dedup.py -v  # single file

# Lint
ruff check .                              # rules: E/F/W/I/B/C4/UP/N
ruff check --fix .                        # auto-fix

# Coverage
coverage run -m pytest tests/ -v && coverage report
```

## Coding Style & Naming Conventions

- **Python**: Chinese docstrings on every function; every `if`-branch commented with business logic; sections delimited by `# ====` banner comments.
- **Frontend**: Vanilla JS, no build tools. `$ (id)` shorthand, `api(method, path, body)` central fetch wrapper with 120s timeout, view-caching router via `navToView()`.
- **Linting**: Ruff with select `E/F/W/I/B/C4/UP/N`; ignores `E501/E402/B904/N806/E741/F601`.
- **Database**: Spare columns pattern (`extra1/extra2/extra3`) for schema flexibility; all tables use `datetime('now', 'localtime')` for timestamps.

## Testing Guidelines

- Framework: **pytest** with `asyncio_mode=auto`. 8 test files, ~245 tests.
- Isolation: `tests/conftest.py` sets `TRADE_HOME` to a temp directory before imports — no real data touched.
- Database: monkeypatch `_get_db_path` for temporary SQLite databases.
- Newly added: `tests/test_customer_dedup.py` — tests for dedup, `compute_data_completeness()`, `build_briefing()`, `health_audit()`.
- Run: `python -m pytest tests/ -v`

## Key Design Points (Recent Changes)

1. **20 skills** (was 19): added `b2b-cold-outreach` for B2B cold outreach emails (development letters, promotion, follow-up).
2. **Brand Safety Guardrails**: `BRAND_SAFETY_BLOCK` in `trade/prompt.py` prohibits derogatory language, fabricated certifications, hype claims, and competitor attacks. Loaded per-company via `get_brand_safety()` or falls back to code default. Injected into system prompt in `build_query()`.
3. **Customer Dedup & Health** (Close.com blog optimizations):
   - Soft dedup warnings on `create()` for email/website duplicates (doesn't block).
   - `bulk_save()` 3-dimensional dedup: name + email + website.
   - `find_duplicates()` — email exact match + website domain-normalized match.
   - `compute_data_completeness()` — weighted 0–100 score across 16 fields.
   - `build_briefing()` — AI customer briefing with identity, contacts, history, orders, completeness.
   - `health_audit()` — detects stale customers, high-value unconverted, and incomplete data.
4. **Conversation Rating**: `POST /conversations/{id}/rate` (1–5 + feedback), stored via `json_set` atomic UPDATE to avoid race conditions. SSE stream returns `conversation_id` in response event for immediate frontend rating.
5. **New API endpoints**: `GET /customers/duplicates`, `GET /cron/health`.

## Commit & Pull Request Guidelines

- **Commits**: Chinese-language conventional commits (`feat:`, `fix:`, `chore:`, `docs:`, `style:`). Keep scoped and atomic.
- **PRs**: Link related issues; include before/after test results; screenshots for UI changes. Ensure `ruff check .` and `python -m pytest tests/ -v` pass.

## Code Annotation Standards

Every function must have a Chinese docstring. Every if-branch must have a comment explaining the business logic. Complex list/dict comprehensions should be split with inline comments. Sections separated by banner comments (`# ====`).

---
> Source: [chefroger/smart-trade-ai](https://github.com/chefroger/smart-trade-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
