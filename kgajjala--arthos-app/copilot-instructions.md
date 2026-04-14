## arthos-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arthos is a Python investment analysis web application built with FastAPI, SQLModel, and yfinance. It provides stock data analysis with technical indicators (SMAs, standard deviation signals), watchlist management, options analysis (covered calls, risk reversals), and interactive candlestick charts.

## Common Commands

### Running the Application
```bash
python run.py
# or
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### Testing

**CRITICAL: ALWAYS use Docker for local testing. Do NOT run pytest directly.**

#### Why Docker Only?
- **Database consistency**: Docker uses PostgreSQL (same as production), not SQLite
- **Migrations tested**: Ensures all migrations work with PostgreSQL
- **Environment parity**: Matches production environment exactly
- **Prevents "works on my machine"**: SQLite vs PostgreSQL differences won't surprise you in production

#### Running Tests
```bash
# MANDATORY: Run ALL tests in Docker before committing
./scripts/test/run-tests-local.sh all

# Run unit tests only (faster, skips browser tests)
./scripts/test/run-tests-local.sh unit

# Run specific test file in Docker
./scripts/test/run-tests-local.sh tests/test_specific.py
```

#### ❌ DO NOT DO THIS (Direct pytest without Docker)
```bash
# ❌ WRONG - Uses SQLite, not PostgreSQL
pytest tests/test_specific.py -v

# ❌ WRONG - Migrations may not match production
pytest tests/ -v -k "not browser and not e2e"

# ❌ WRONG - Local environment differs from production
python -m pytest tests/
```

#### When to Use Direct pytest (Rare Exceptions)
Only use direct pytest for:
- Quick syntax checks during development
- Debugging a specific test failure (after Docker tests fail)
- **BUT**: Always re-run in Docker before committing

#### Docker Test Commands
```bash
# Clean start (removes old containers/volumes)
docker-compose -f docker-compose.test.yml down -v
./scripts/test/run-tests-local.sh all

# Run with random order to detect test dependencies
./scripts/test/run-tests-local.sh all --random-order
```

#### Test Suite Performance Benchmarks
**Current performance (as of Feb 2026):**
- **Full suite**: ~3 minutes 15 seconds (364 tests including browser tests)
  - Unit tests: ~40 seconds (quick feedback loop)
  - Integration + API tests: ~1 minute
  - Browser tests (Playwright): ~1 minute 30 seconds
- **Test breakdown**: 364 passed, 3 skipped, 28 warnings (deprecations)

**Why this matters:**
- Sub-4-minute full suite enables rapid iteration
- Developers can run full suite multiple times per hour
- Browser tests ensure UI changes don't break existing functionality
- PostgreSQL in Docker ensures production parity

**Performance regression monitoring:**
- If full suite exceeds 5 minutes, investigate slow tests
- Use `pytest --durations=10` to identify slowest tests
- Consider parallelization if suite grows beyond 500 tests

### Pre-Push Workflow
```bash
docker-compose -f docker-compose.test.yml up test-runner --abort-on-container-exit
```

## Token Optimization Guidelines

**CRITICAL**: Follow these guidelines to minimize token usage during development.

### General Principles
- **Read only what you need**: Use targeted file reads instead of reading entire large files
- **Limit log fetches**: When checking production logs, always use `| tail -N` or `| head -N` (default: 100 lines max)
- **Run targeted tests**: Run only new/modified tests during development, full suite before commit
- **Avoid redundant reads**: If you've already read a file in the session, refer to it instead of re-reading

### Production Log Fetching
```bash
# ✅ CORRECT - Limit to 100 lines (conserves tokens)
railway logs --json | tail -100

# ✅ CORRECT - Limit to 50 lines for quick check
railway logs --json | tail -50

# ❌ WRONG - Fetches all logs (wastes tokens)
railway logs --json
```

### Test Running Strategy
```bash
# During development - run ONLY new/modified tests
docker-compose -f docker-compose.test.yml run --rm test-runner pytest tests/test_new_feature.py -v

# Before commit - run full suite (MANDATORY)
./scripts/test/run-tests-local.sh all

# Smart test runner - automatically selects affected tests
./scripts/test/smart-test-runner.sh
```

### File Reading Strategy
- **Before reading large files**, check line count: `wc -l <file>`
- **Use `limit` and `offset` parameters** in Read tool for large files
- **Read specific sections** instead of entire templates (see refactoring plan below)

## Code Organization Patterns

### Modular Architecture (Completed Feb 2026)

**As of Feb 2026, the codebase has been refactored for token efficiency and maintainability:**

#### Router Organization
- **Main file**: `app/main.py` (237 lines) - Core FastAPI setup, middleware, includes
- **Router modules** (in `app/routers/`):
  - `watchlist_routes.py` - Watchlist CRUD operations
  - `notes_routes.py` - Stock notes management
  - `insights_routes.py` - LLM insights and model management
  - `stock_routes.py` - Stock detail pages and API endpoints
  - `rr_routes.py` - Risk reversal watchlist management
  - `debug_routes.py` - Debug tools and admin pages

#### Service Organization
- **Core services** (in `app/services/`):
  - `stock_data_service.py` - Raw data fetching from providers
  - `options_data_service.py` - Options chain processing and LEAPS
  - `covered_call_service.py` - Covered call strategy calculations
  - `risk_reversal_service.py` - Risk reversal strategy calculations
  - `stock_service.py` - Metric calculations and signal generation (retained)

#### Utility Modules
- **`app/utils/route_helpers.py`** - Shared utility functions for routes:
  - `_require_admin()` - Admin access checking
  - `_format_duration()` - Time formatting for UI

### Code Organization Guidelines

#### When to Create New Router Modules

**Create a new router module when:**
- A feature has 5+ related endpoints
- Routes serve a distinct domain (watchlists, notes, insights)
- You want to isolate feature-specific logic for token efficiency

**Pattern:**
```python
# app/routers/feature_routes.py
from fastapi import APIRouter, Request, HTTPException
from fastapi.responses import HTMLResponse
from app.utils.route_helpers import _require_admin  # Import shared utilities

router = APIRouter()

@router.get("/v1/feature/{id}")
async def get_feature(id: str):
    # Implementation
    pass

@router.post("/v1/feature")
async def create_feature(data: FeatureCreate):
    # Implementation
    pass
```

**Then include in main.py:**
```python
from app.routers import feature_routes
app.include_router(feature_routes.router)
```

#### When to Create New Service Modules

**Create a new service module when:**
- A service file exceeds 500 lines
- You can extract a cohesive subset of functionality (e.g., "options data" vs "covered calls")
- The extracted module has a clear single responsibility

**Pattern:**
```python
# app/services/feature_service.py
from typing import Dict, Any, List
import logging

logger = logging.getLogger(__name__)

def process_feature_data(input: Any) -> Dict[str, Any]:
    """
    Process feature data with clear documentation.

    Args:
        input: Description of input

    Returns:
        Dictionary with processed data
    """
    # Implementation
    pass
```

**Import in dependent services:**
```python
# app/services/higher_level_service.py
from app.services.feature_service import process_feature_data

def higher_level_function():
    result = process_feature_data(data)
    # Use result
```

#### Handling Circular Dependencies

**Problem:** Service A needs Service B, and Service B needs Service A

**Solution: Use late imports inside functions**
```python
# app/services/stock_metrics_service.py
def get_multiple_stock_metrics(tickers: List[str]) -> List[Dict[str, Any]]:
    """Get metrics for multiple tickers."""
    # Import only when function is called, not at module load time
    from app.services.stock_price_service import get_stock_metrics_from_db

    results = []
    for ticker in tickers:
        try:
            metrics = get_stock_metrics_from_db(ticker)
            results.append(metrics)
        except ValueError:
            # Handle error
            pass
    return results
```

**Document circular dependencies:**
```python
# At top of file
"""
Stock metrics calculation service.

Note: Uses late import of stock_price_service to avoid circular dependency.
"""
```

#### Utility Modules for Shared Code

**Create utility modules to avoid code duplication:**

```python
# app/utils/route_helpers.py
"""Shared helper functions for route handlers."""
import os
from fastapi import HTTPException, Request

def _require_admin(request: Request):
    """
    Check that the request comes from an admin user.

    Raises:
        HTTPException: If user is not admin
    """
    user = request.session.get("user") if hasattr(request, "session") else None
    if not user:
        raise HTTPException(status_code=403, detail="Admin access required")

    admin_email = os.getenv("ADMIN_EMAIL")
    if not admin_email or user.get("email") != admin_email:
        raise HTTPException(status_code=403, detail="Admin access required")
```

**When to create utility modules:**
- Function is used in 3+ different modules
- Function is truly generic (not domain-specific)
- Function has no external dependencies (or minimal ones)

#### Template Organization (Future)

**Current state:** `app/templates/stock_detail.html` (1,493 lines) - Not yet extracted

**Recommended pattern when extracting:**
```
app/templates/stock/
├── detail_header.html       # Price metrics cards
├── chart_section.html       # Plotly chart
├── insights_tab.html        # AI insights tab
├── covered_calls_tab.html   # Covered calls table
├── risk_reversal_tab.html   # RR strategies table
└── notes_tab.html           # Notes management
```

**Use Jinja2 includes:**
```jinja2
<!-- In stock_detail.html -->
{% include 'stock/detail_header.html' %}
{% include 'stock/chart_section.html' %}

<!-- In tab content -->
<div class="tab-pane fade" id="insights">
  {% include 'stock/insights_tab.html' %}
</div>
```

**Note:** Template extraction is lower priority - templates are edited less frequently than Python code.

### Token Optimization Achieved

**Phase 1 - Router Extraction (Complete):**
- `main.py`: 1,843 → 237 lines (87% reduction)
- Token savings: ~80% when reading feature-specific routes
- Example: Reading notes routes (122 lines) vs old main.py (1,843 lines)

**Phase 2 - Service Extraction (80% Complete):**
- Extracted: stock_data, options_data, covered_call, risk_reversal services
- Deferred: stock_metrics (kept in stock_service.py)
- Token savings: ~75% when reading specific strategy logic

**Phase 3 - Template Extraction (Deferred):**
- Can be done incrementally when working on UI features
- Lower priority as templates are edited less frequently

### Guidelines for New Code

**When adding new features:**

1. **For new API endpoints:**
   - If related to existing router (e.g., watchlists), add to that router
   - If new domain area, create new router module in `app/routers/`

2. **For new business logic:**
   - If related to existing service, add to that service
   - If service exceeds 500 lines after addition, consider extracting
   - Create new service module if it's a distinct responsibility

3. **For shared utilities:**
   - Add to `app/utils/route_helpers.py` if route-related
   - Create new utility module if different domain (e.g., `app/utils/formatting.py`)

4. **For new templates:**
   - Keep simple templates as single files
   - Consider extraction if template exceeds 500 lines
   - Use Jinja2 includes for repeated sections (navbar, footer, etc.)

## Development Workflow

**CRITICAL**: All code changes MUST follow this workflow. No exceptions.

### Core Principle: Finish What You Start

**As a good developer, when you run into issues, you must finish your task and go back and figure out the issues and fix it.**

- When tests fail, **fix them immediately** before moving on
- When scripts encounter errors, **debug and resolve them** before proceeding
- **Bring tasks to logical conclusion** - don't leave the repo in a dirty state
- If you discover issues during implementation:
  1. Fix the issue
  2. Update tests if needed
  3. Re-run affected tests (not full suite during development)
  4. Run full suite before commit
  5. Only then commit and push
- **Never skip issues** with a note to "fix later" - fix them now

Examples:
- ✅ Test fails → Fix test → Re-run affected tests → Verify pass → Commit (after full suite)
- ❌ Test fails → Comment it out → Commit (leaves broken tests)
- ✅ Script error → Debug script → Fix root cause → Verify → Commit
- ❌ Script error → "We'll fix that later" → Move on (leaves broken tooling)

### 1. Make Changes & Add Tests
- **All code changes require test cases** (excludes documentation/config files)
- When adding new functionality:
  - Backend changes: Add unit/integration tests
  - New API endpoints with UI: Add Playwright browser tests
  - Backend-only API endpoints: Add API/integration tests only
  - UI changes (new pages, interactions, styling): Add/update/remove Playwright tests
- **When modifying existing behavior:**
  - **CRITICAL**: If you change UI behavior (tab order, default values, element visibility, text content, etc.), you MUST update ALL tests that verify the old behavior
  - Example: Changing default active tab from "Covered Calls" to "Insights" requires updating every test that assumes "Covered Calls" is default
  - Example: Changing button text from "View My WatchLists" to "WatchLists" requires updating all tests checking for that text
  - **Don't assume tests will pass** - UI changes almost always require test updates
- **Review code for simplicity**: Before writing tests, refactor code to make it testable and simple
  - Simpler code = simpler tests = easier maintenance

### 2. Run Tests in Docker

#### Fast Iteration (During Development)
```bash
# Run smart test runner - automatically selects affected tests based on changes
./scripts/test/smart-test-runner.sh

# Or run specific test file you're working on
docker-compose -f docker-compose.test.yml run --rm test-runner pytest tests/test_watchlist_service.py -v
```

**Smart test runner benefits:**
- **Fast feedback**: ~40s to 2 min (vs 3.5 min full suite)
- **Intelligent selection**: Analyzes git changes to run affected tests
- **Always includes smoke tests**: Catches critical regressions
- **Use during active development** for rapid iteration

#### Full Suite (Before Commit - MANDATORY)
```bash
# MANDATORY: Run ALL tests before pushing to main
./scripts/test/run-tests-local.sh all
```

- **Why all tests?** Backend changes can affect UI, so we always run the full suite before commit
- **Why Docker?** Uses PostgreSQL (same as production), not SQLite - ensures migrations work
- Tests MUST pass 100% before proceeding to push
- **When tests fail after your changes:**
  1. **Analyze the failure**: Is it a bug in your code or expected due to intentional behavior change?
  2. **If it's a bug**: Fix your code and re-run affected tests only
  3. **If it's expected** (you changed UI behavior): Update the failing tests to match new expected behavior
  4. **Search for similar tests**: If you changed "Covered Calls" tab behavior, search codebase for all tests mentioning "covered-calls" and update them
  5. **Re-run affected tests** after fixing, then run full suite once before commit
- **Never run pytest directly** - always use Docker for consistency with production

#### Recommended Workflow (Token-Optimized)
1. **During development**: Run ONLY new/modified tests you're actively working on
2. **After completing feature**: Run smart test runner to catch affected tests
3. **Before committing**: Run full suite ONCE (mandatory)
4. **Never run full suite multiple times** unless fixing test failures

### 3. Generate Test Metrics Report

**MANDATORY**: Before committing, generate and review test metrics.

```bash
# Run test metrics reporter with smart test runner
./scripts/test/test-metrics-reporter.sh

# Or with full suite
./scripts/test/test-metrics-reporter.sh "all"

# Or with custom command
./scripts/test/test-metrics-reporter.sh "pytest tests/test_watchlist*.py"
```

**The reporter automatically provides:**
1. ✨ New tests added
2. 📝 Existing tests updated
3. 🗑️ Tests removed
4. 🎯 Tests selected for execution
5. ⏱️ Time taken to run selected tests
6. 💰 Time saved vs full suite

**Include metrics in commit message** - See `.github/TEST_METRICS_TEMPLATE.md` for format.

### 4. Commit Changes
```bash
git add <files>
git commit -m "Descriptive message

Details about the change...

Test Metrics:
- New tests: X, Updated: Y, Removed: Z
- Ran: N tests in Xs (saved Ys vs full suite)

Co-Authored-By: Claude <noreply@anthropic.com>"
```
- Bundle related changes into a single commit (implementation + tests + fixes)
- Use descriptive commit messages with context
- **ALWAYS include test metrics** in commit message

### 5. Push to Main
```bash
git push origin main
```
- Push immediately after local commit
- Never leave commits unpushed

### 6. Wait for Deployment
- Monitor Railway deployment status
- Wait for deployment to complete before verification

### 7. Verify in Production
**Production URL:** `https://my.arthos.app`

**Quick automated verification:**
```bash
# Run automated verification script
./scripts/verify_production.sh
```

**Required verification steps (automated by Claude):**

a. **Run automated verification script**
   - Checks site accessibility (https://my.arthos.app)
   - Scans Railway logs for errors
   - Verifies critical endpoints (home, watchlists, etc.)
   - Reports all issues found

b. **Check Railway logs for errors**
   ```bash
   # ALWAYS limit logs to conserve tokens (default: 100 lines)
   /opt/homebrew/bin/railway logs --json | tail -100

   # For quick check, use 50 lines
   /opt/homebrew/bin/railway logs --json | tail -50
   ```
   - Scan for errors that only appear in production
   - Look for stack traces, 500 errors, database errors
   - Check for any unexpected warnings
   - Report any issues found
   - **NEVER fetch unlimited logs** - always use tail/head to limit output

c. **Manual functional verification** (User or Claude with Playwright)
   - For UI changes: Manually test changed functionality
   - For backend changes: Verify API endpoints work
   - For Playwright automation (future):
     ```bash
     TEST_SERVER_URL=https://my.arthos.app pytest tests/test_*_browser.py -v
     ```

d. **Monitor error tracking**
   - Check Sentry (if configured) for new errors
   - Verify no new exceptions were introduced

**Test Account Setup:**
- **Email:** `arthos.test@gmail.com`
- **Credentials:** Stored in `.env` file (gitignored)
  - `TEST_GOOGLE_EMAIL=arthos.test@gmail.com`
  - `TEST_GOOGLE_PASSWORD=<secure-password>`
- **Initial setup** (one-time):
  1. Log in manually to https://my.arthos.app with test account
  2. Complete Google OAuth flow
  3. Run test data setup script:
     ```bash
     # Get production DATABASE_URL from Railway
     export DATABASE_URL=<production-url>
     python scripts/setup_production_test_data.py
     ```
- **Used for:** Automated Playwright testing against production

**Running Tests Against Production:**
```bash
# Run all browser tests against production
TEST_SERVER_URL=https://my.arthos.app pytest tests/test_*_browser.py -v

# Run specific browser test
TEST_SERVER_URL=https://my.arthos.app pytest tests/test_watchlist_browser.py::TestWatchListBrowser::test_create_watchlist -v

# Cleanup test data in production
export DATABASE_URL=<production-url>
python scripts/setup_production_test_data.py --cleanup
```

### 7. Fix Production Issues (if any)
If errors are found:
1. Investigate and fix locally
2. Test the fix locally (Docker tests must pass)
3. Commit and push to main
4. Verify fix in production
5. Repeat until production is clean

### Exceptions
- **Documentation-only changes**: Skip testing, push directly
- **Config files outside code**: Skip testing (e.g., `.gitignore`, `README.md`)
- **Emergency hotfixes**: Follow same workflow (will revisit fast-track process in future)

### Quick Reference
```bash
# Complete workflow in one go:
./scripts/test/run-tests-local.sh all && \
git add -A && \
git commit -m "Your message" && \
git push origin main && \
echo "Now verify in production!"
```

## Architecture

### Data Provider Pattern
The app uses an abstraction layer for stock data providers:
- `app/providers/base.py` - Abstract `StockDataProvider` class with standardized data classes (`StockPriceData`, `StockInfo`, `OptionQuote`, `OptionsChain`)
- `app/providers/yfinance_provider.py` - YFinance implementation
- `app/providers/marketdata_provider.py` - MarketData API implementation
- `app/providers/factory.py` - Factory for getting the configured provider

### Key Services

**Data Layer:**
- `app/services/stock_price_service.py` - Database-driven stock price storage and retrieval
- `app/services/stock_data_service.py` - Raw data fetching from providers (yfinance, MarketData)

**Analysis Layer:**
- `app/services/stock_service.py` - Metric calculations (SMAs, signals, devstep)
- `app/services/options_data_service.py` - Options chain processing and LEAPS expirations
- `app/services/covered_call_service.py` - Covered call strategy calculations
- `app/services/risk_reversal_service.py` - Risk reversal strategy calculations

**Business Logic Layer:**
- `app/services/watchlist_service.py` - Watchlist CRUD operations
- `app/services/rr_watchlist_service.py` - Risk reversal watchlist tracking
- `app/services/insights_service.py` - LLM-powered stock insights

**Background Jobs:**
- `app/services/scheduler_service.py` - Periodic stock data fetching (every 60 minutes)

### Models (SQLModel)
- `app/models/stock_price.py` - `StockPrice` (daily OHLC) and `StockAttributes` (ticker metadata, dividends, earnings)
- `app/models/watchlist.py` - `WatchList` and `WatchListStock`
- `app/models/rr_watchlist.py` - Risk reversal tracking
- `app/models/account.py` - User accounts (OIDC/Google auth)

### Database
- **PostgreSQL for both local development and production** (consistency is critical)
- Local dev: `docker-compose -f docker-compose.dev.yml up -d` then set `DATABASE_URL=postgresql://arthos:arthos_dev@localhost:5432/arthos`
- Production: Railway PostgreSQL (set `DATABASE_URL` environment variable)

### Table Naming
- **Always verify table names from model `__tablename__`** - don't assume based on patterns
- Table names are inconsistent: `watchlist_stocks` (plural) vs `stock_price` (singular)
- When writing raw SQL, check the model definition first:
  - `StockPrice` → `stock_price`
  - `StockAttributes` → `stock_attributes`
  - `WatchListStock` → `watchlist_stocks`
  - `WatchList` → `watchlist`
  - `LLMModel` → `llm_model`
  - `AppSettings` → `app_settings`

### UUID Handling
- **Always use UUID type consistently** - both in models and database columns
- All ID fields (watchlist_id, account_id, rr_uuid) should use UUID type in PostgreSQL
- When accepting UUID parameters in service functions, use `Union[UUID, str]` type hints
- Always convert to string using `to_str()` from `app/utils/type_helpers.py` before database operations
- This prevents "operator does not exist: character varying = uuid" errors in PostgreSQL

## Testing Guidelines

### Test Data Management
- **Never rely on external APIs (yfinance) in tests** - use `populate_test_stock_prices()` from `tests/conftest.py`
- Clean up database BEFORE populating test data to avoid UNIQUE constraint violations
- Use `autouse=True` fixtures for consistent setup/cleanup

### Model Attribute Names
Use correct attribute names from model definitions:
- `watchlist.watchlist_id` (not `watchlist.id`)
- `StockPrice.price_date`, `StockPrice.close_price`

### Playwright Browser Tests
- Scope locators to specific sections: `page.locator("#option-data th:has-text('Strike')").first`
- Use `.first` for duplicate elements to avoid strict mode violations
- Set up dialog handlers BEFORE clicking buttons that trigger them

### Common Pitfalls
- `pd.isinf()` doesn't exist - use `np.isinf()` from numpy
- Normalize timestamps to timezone-naive: `df.index.tz_localize(None)`
- **Template conditionals with enums/ratios**: When displaying badges or labels based on enum-like values (e.g., RR ratios: '1:1', '1:2', '1:3', 'Collar'), ensure ALL valid values are handled. Don't use `else` as a catch-all assuming only one default - explicitly check each value to avoid incorrect displays.
- **Consistent styling across related pages**: When the same data (e.g., ratio badges) appears on multiple pages (list view and detail view), ensure colors/styling are consistent. Check existing pages for the established color scheme before adding new styles.

### Database Migrations
**CRITICAL**: When adding new columns to existing SQLModel models, you MUST add migrations in BOTH places:

#### 1. Add Migration to `app/database.py` (Required for Local Dev + Docker Tests)
- `create_db_and_tables()` only creates NEW tables, it does NOT add columns to existing tables
- Add a migration function following existing patterns (e.g., `_migrate_stock_attributes_insights_columns()`)
- Call the migration in `create_db_and_tables()` in the appropriate order
- **MUST support both SQLite (local dev) and PostgreSQL (production/Docker tests)**

#### 2. Add Migration to `railway_deploy.py` (Required for Production)
- Production uses flag files to ensure one-time migrations don't run twice
- Add migration function with flag file (e.g., `ADD_INSIGHTS_COLUMNS_FLAG`)
- Add to `main()` function to run on deployment
- Use `AUTOCOMMIT` mode for DDL operations

#### Migration Pattern Example:
```python
# In app/database.py - add to create_db_and_tables()
def _migrate_table_new_columns():
    """Add new_column to table if it doesn't exist."""
    try:
        with Session(engine) as session:
            is_sqlite = DATABASE_URL.startswith("sqlite")

            if is_sqlite:
                # SQLite: Check column exists
                result = session.exec(text("PRAGMA table_info(table_name)")).all()
                column_exists = any(row[1] == 'new_column' for row in result)
                if not column_exists:
                    session.exec(text("ALTER TABLE table_name ADD COLUMN new_column TEXT"))
                    session.commit()
                    print("Added new_column to table_name")
            else:
                # PostgreSQL: Check column exists
                result = session.exec(text(
                    "SELECT column_name FROM information_schema.columns "
                    "WHERE table_name = 'table_name' AND column_name = 'new_column'"
                )).first()
                if not result:
                    session.exec(text("ALTER TABLE table_name ADD COLUMN new_column TEXT"))
                    session.commit()
                    print("Added new_column to table_name")
    except Exception as e:
        print(f"Warning: Could not migrate table: {e}")

# Then add to create_db_and_tables():
    _migrate_table_new_columns()
```

#### Testing Checklist:
- ✅ Test with SQLite (local dev): Delete `arthos.db` and restart app
- ✅ Test with Docker PostgreSQL: `./scripts/test/run-tests-local.sh all`
- ✅ Verify production deployment doesn't break (Railway runs `railway_deploy.py`)
- ✅ Check that both SQLite and PostgreSQL code paths work

#### Why Two Files?
- **`app/database.py`**: Called on EVERY app startup (local, Docker, production) - no flag files
- **`railway_deploy.py`**: Production-only with flag files for expensive one-time operations (backfills, data fixes)

## UI Standards

### Color Coding for Financial Data
- **Prices (NO color)**: Entry Price, Current Price, Net Cost, Strike, Premiums - display without color, use minus sign for negatives
- **Changes/Returns (WITH color)**: Change, Change %, Total Return - green for positive, red for negative

## API Structure

### Stock Endpoints
- `GET /v1/stock?q={ticker}` - Get stock metrics (SMA, signal, price)
- `GET /stock/{ticker}` - Stock detail page with chart
- `GET /results?tickers={csv}` - Multi-ticker results page

### Watchlist Endpoints
- `GET/POST /v1/watchlist` - List/create watchlists
- `GET/PUT/DELETE /v1/watchlist/{id}` - Watchlist CRUD
- `POST /v1/watchlist/{id}/stocks` - Add stocks
- `DELETE /v1/watchlist/{id}/stocks/{ticker}` - Remove stock

### Debug Endpoints (development)
- `GET /debug/stock-price?ticker={ticker}` - View stock price data
- `GET /debug/scheduler-log` - View scheduler logs
- `POST /debug/scheduler-log/trigger` - Manually trigger scheduler

## Environment Variables

Required environment variables for production:
- `DATABASE_URL` - PostgreSQL connection string
- `SECRET_KEY` - Session secret for authentication
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` - OAuth for Google login
- `OPENROUTER_API_KEY` - API key for OpenRouter (required for LLM insights)
- `OPENROUTER_MODEL` - (optional) Fallback model if DB has no models configured, defaults to `anthropic/claude-3.5-sonnet`
- `SENTRY_DSN` - (optional) Error tracking

## Deployment Notes

### Railway Deployment
- Deploy script: `railway_deploy.py` runs migrations before app starts
- Logs: `railway logs --json` to view deployment/runtime logs
- Redeploy: `railway redeploy --yes` (may fail if build in progress)
- Cache issues: Add a cache bust comment to `requirements.txt` if Railway uses stale builds

### Python Dependencies
- **Google Generative AI SDK**: Don't pin `protobuf` version - let pip resolve it automatically. The SDK has specific protobuf requirements (`<6.0.0`) that conflict with newer versions.
- Always test dependency changes locally before deploying

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kgajjala)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kgajjala)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
