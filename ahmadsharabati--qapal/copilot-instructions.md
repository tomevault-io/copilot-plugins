## qapal

> **QAPAL** is a deterministic AI-powered UI test automation framework written in Python. The core philosophy is:

# CLAUDE.md — AI Assistant Guide for QAPAL

## Project Overview

**QAPAL** is a deterministic AI-powered UI test automation framework written in Python. The core philosophy is:

> **AI plans once. Code executes deterministically.**

The system separates AI-driven test planning from code-based execution, achieving ~87.5% token savings compared to traditional AI-in-the-loop approaches. After a one-time planning phase, test execution runs without any AI calls.

---

## Repository Structure

```
QAPAL/
├── main.py              # CLI orchestrator (entry point)
├── executor.py          # Deterministic test runner (~54K, largest file)
├── assertions.py        # Assertion registry (~45K, 30+ assertion types)
├── actions.py           # Action registry (~42K, 19 action types)
├── generator.py         # PRD → test plan generator
├── crawler.py           # Page crawler + element extraction
├── locator_db.py        # TinyDB wrapper for persistent locator state
├── planner.py           # Lower-level test planner (maps steps to selectors)
├── state_graph.py       # Navigation graph mapper
├── ai_client.py         # Unified AI provider wrapper (Claude/GPT-4/Grok)
├── replanner.py         # Self-healing recovery logic
├── semantic_extractor.py # Page understanding / semantic context
├── smoke_semantic.py    # Diagnostic utility for semantic extraction
├── __init__.py          # Package exports
├── requirements.txt     # Dependencies
├── tests/
│   ├── test.py          # Unit tests (mocked, no network)
│   └── test_e2e.py      # End-to-end integration tests
├── plans/               # 59 generated JSON test plans (TC*.json)
└── *.md                 # PRD documents for various test scenarios
```

---

## Three-Phase Architecture

### Phase 1: Crawling (`crawler.py`, `locator_db.py`)
- Playwright crawls live pages and extracts interactive elements
- Two extraction strategies:
  1. **Accessibility tree** (primary) — ARIA roles, semantic elements
  2. **DOM scan** (secondary) — `data-testid`, `onclick` handlers, `tabindex`
- Stores results in `locators.json` (TinyDB) keyed by `url + role + name + container + frame_url + dom_path`
- Stale-check: skips re-crawl if data is less than 60 minutes old

### Phase 2: Planning (`generator.py`, `planner.py`, `state_graph.py`)
- Reads a PRD (Markdown file) + available locators from DB
- Makes a **single AI call** to produce a frozen JSON execution plan
- Output: all selectors, actions, and assertions predefined — no runtime AI needed
- State graph (`state_graph.py`) captures navigation topology to aid multi-page planning

### Phase 3: Execution (`executor.py`)
- Runs the frozen JSON plan **without any AI calls** under normal conditions
- Locator resolution chain (in order):
  1. Primary selector from plan
  2. Fallback selector from plan
  3. AI rediscovery (one emergency call if element not found)
  4. Hard fail with screenshot
- Self-healing via `replanner.py`: triggered when executor hits an unexpected state
- Hard caps: 1 replan per test, 3 unknown states per test run

---

## Key Conventions

### Selector Strategies (priority order)

Always prefer more stable selectors. The priority is:

1. `testid` — `data-testid` attribute (most reliable)
2. `role` — ARIA role + accessible name
3. `role_container` — scoped role + name within a parent
4. `aria_label` — explicit `aria-label` attribute
5. `placeholder` — input placeholder text
6. `testid_prefix` — prefix-based matching for list items/cards
7. `text` — visible text content
8. `label` — form label text
9. `css` — CSS selector (fragile, last resort)

Never use dynamic IDs, numeric indexes, or generated class names in plans.

### Plan JSON Format

```json
{
  "test_id": "TC001_login",
  "name": "User can log in successfully",
  "steps": [
    {"action": "navigate", "url": "https://app.com/login"},
    {"action": "fill", "selector": {"strategy": "testid", "value": "email"}, "value": "user@example.com"},
    {"action": "click", "selector": {"strategy": "testid", "value": "login-submit"}}
  ],
  "assertions": [
    {"type": "url_equals", "value": "https://app.com/account"}
  ],
  "_meta": {
    "source": "prd_generator",
    "planned_at": "2026-03-12T...",
    "locators": 347,
    "ai_model": "..."
  }
}
```

### PRD Format

PRD files (Product Requirements Documents) are Markdown files that describe test scenarios. They follow the pattern in `sample_prd.md`. Each test case should describe:
- The user goal
- Preconditions
- Step-by-step interactions
- Expected outcomes

The generator converts these to JSON plans automatically.

---

## Environment Variables

All configuration is via environment variables — no static config files.

```bash
# AI Provider (required — pick one)
QAPAL_AI_PROVIDER=anthropic    # anthropic | openai | grok
ANTHROPIC_API_KEY=...          # For Claude (Anthropic)
OPENAI_API_KEY=...             # For GPT-4 (OpenAI)
XAI_API_KEY=...                # For Grok (xAI)
QAPAL_AI_BASE_URL=...          # For custom OpenAI-compatible endpoints

# Storage
QAPAL_DB_PATH=locators.json    # Path to TinyDB store (default: locators.json)

# Browser
QAPAL_HEADLESS=true            # Run browser headlessly (default: true)
QAPAL_SCREENSHOTS=reports/screenshots  # Screenshot output directory

# Timeouts (milliseconds)
QAPAL_ACTION_TIMEOUT=10000     # Default: 10000ms
QAPAL_ASSERTION_TIMEOUT=5000   # Default: 5000ms

# Self-Healing / Recovery
QAPAL_AI_REDISCOVERY=true      # Enable AI fallback for missing locators
QAPAL_MAX_REPLANS=1            # Max recovery attempts per test (default: 1)
QAPAL_MAX_UNKNOWN_STATES=3     # Max unexpected states before hard fail
QAPAL_MAX_URL_VISITS=3         # Max times to visit same URL (default: 3)

# Crawling
CRAWLER_STALE_MINUTES=60       # Skip re-crawl if < N minutes old (default: 60)
QAPAL_CRAWL_CONCURRENCY=3      # Parallel crawl workers (default: 3)
```

Create a `.env` file in the project root — it is auto-loaded via `python-dotenv`. The `.gitignore` excludes `.env`.

---

## CLI Commands

All commands are invoked via `main.py`:

```bash
# Crawl a URL and store locators
python main.py crawl --url https://example.com

# Generate a test plan from a PRD
python main.py plan --prd path/to/prd.md --url https://example.com

# Run an existing test plan
python main.py run --plan plans/TC001_login.json

# Show locator DB status
python main.py status

# Run full pipeline: crawl + plan + run from a PRD
python main.py prd-run --prd path/to/prd.md --url https://example.com

# Build / view state navigation graph
python main.py graph --url https://example.com
```

---

## Supported Actions (19 total)

Defined in `actions.py` — single source of truth for AI and executor:

| Category | Actions |
|----------|---------|
| Navigation | `navigate`, `refresh`, `go_back`, `go_forward` |
| Interaction | `click`, `dblclick`, `hover`, `scroll` |
| Input | `fill`, `type`, `clear`, `press`, `select` |
| State | `check`, `uncheck`, `focus`, `blur` |
| Utility | `wait`, `screenshot`, `evaluate` |

---

## Supported Assertions (30+ types)

Defined in `assertions.py` — single source of truth for AI and executor:

| Category | Assertion Types |
|----------|----------------|
| URL | `url_equals`, `url_contains`, `url_matches` |
| Page | `title_equals`, `title_contains` |
| Existence | `element_exists`, `element_visible`, `element_hidden` |
| State | `element_enabled`, `element_disabled`, `element_checked`, `element_unchecked`, `element_focused`, `element_editable`, `element_readonly`, `element_in_viewport` |
| Content | `element_text_equals`, `element_text_contains`, `element_text_matches`, `element_value_equals`, `element_value_contains` |
| Attributes | `element_attribute`, `element_has_class`, `element_has_style`, `element_count` |
| Custom | `javascript` |

---

## Dependencies

Install with:
```bash
pip install -r requirements.txt
playwright install chromium
```

Key dependencies:
- `playwright>=1.40.0` — Browser automation
- `tinydb>=4.8.0` — Local JSON document store
- `anthropic>=0.18.0` — Claude API
- `openai>=1.0.0` — GPT-4 API
- `python-dotenv>=1.0.0` — `.env` loading
- `crawl4ai>=0.3.0` — Optional rich semantic extraction (falls back to Playwright a11y)

---

## Running Tests

```bash
# Unit tests only (no network, fully mocked)
python tests/test.py

# All end-to-end tests
python tests/test_e2e.py

# Specific E2E group with pytest
python -m pytest tests/test_e2e.py -k "Crawler"
python -m pytest tests/test_e2e.py -k "Executor"
python -m pytest tests/test_e2e.py -k "Generator"
```

**E2E test URLs used** (real sites, require network):
- `https://demo.playwright.dev/todomvc/#/` — TodoMVC
- `https://books.toscrape.com/` — Book catalog scraping
- `https://playwright.dev/` — Documentation site
- `https://practicesoftwaretesting.com/` — E-commerce (Toolshop)

---

## Development Guidelines

### When modifying `actions.py` or `assertions.py`
- These files are the **contract** between the AI planner and the executor
- Any new action/assertion type added here must be handled in `executor.py`
- Update the docstrings — the AI reads these to know what it can use
- Never remove or rename existing types without updating all plan JSON files in `plans/`

### When modifying `generator.py` or `planner.py`
- Do not add AI calls in loops — the entire plan is generated in a **single AI call**
- The prompt must include: available locators, selector strategies, action/assertion contracts
- Output must be valid JSON parseable by `executor.py`

### When modifying `executor.py`
- Keep execution **stateless** with respect to AI — no AI calls in the happy path
- The locator resolution chain must always end with a hard fail + screenshot, never silent skip
- Respect `QAPAL_ACTION_TIMEOUT` and `QAPAL_ASSERTION_TIMEOUT` from env vars

### When modifying `locator_db.py`
- The identity key formula is: `url + role + name + container + frame_url + dom_path`
- Dynamic name normalization strips prices, timestamps, and counters (important for stability)
- TinyDB is not thread-safe — use `QAPAL_CRAWL_CONCURRENCY` to control parallel crawls

### When modifying `ai_client.py`
- All AI providers must expose the same interface: `complete(messages)` and `acomplete(messages)`
- New providers: add to the factory function, document the required env var

### When adding new PRD files
- Follow the pattern in `sample_prd.md`
- Place them in the project root (alongside existing `*_prd.md` files)
- Store generated plans in `plans/` as `TC<NNN>_<slug>.json`

---

## Design Principles

1. **Determinism** — AI plans once; execution has zero randomness
2. **Token efficiency** — ~500 tokens per test vs. 4,000+ in traditional loops
3. **Separation of concerns** — Planner ≠ Executor ≠ Crawler
4. **Style-blindness** — Assertions ignore visual decorations (bold, strikethrough)
5. **Fallback chains** — Auto-retry with alternative selector strategies
6. **Self-healing** — Replanner recovers mid-test on unexpected states (capped)
7. **Persistent learning** — State graph grows over time, improving future plans
8. **Contract-based** — `actions.py` and `assertions.py` are the single source of truth
9. **Cacheable plans** — JSON plans can be version-controlled, replayed, and audited

---

## Common Pitfalls

- **Never use dynamic IDs** in selectors (e.g., `#item-7482`). Use `testid`, `role`, or `aria_label`.
- **Never add AI calls in the executor hot path** — this breaks the token-efficiency guarantee.
- **Don't hardcode URLs** — use env vars or pass via CLI; plans embed the URL at plan time.
- **locators.json is not committed** (see `.gitignore`) — it's regenerated per environment.
- **Crawl4AI is optional** — the system falls back to Playwright accessibility tree if unavailable.
- **Plans in `plans/` are committed** — they represent the frozen, version-controlled test suite.
- **The `.env` file is never committed** — always use `.env.example` as documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmadsharabati)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ahmadsharabati)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
