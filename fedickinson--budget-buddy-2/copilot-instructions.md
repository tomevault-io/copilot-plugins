## budget-buddy-2

> Critical context for Claude Code to avoid common issues and maintain consistency.

# Claude Development Notes for Budget Buddy

Critical context for Claude Code to avoid common issues and maintain consistency.

---

## 🐘 PostgreSQL Local Development (February 2026)

**MIGRATION COMPLETE**: Budget Buddy now uses PostgreSQL 16 for all environments.

### Quick Reference

**Connection:** `127.0.0.1:5432` / `budget_buddy_dev` / `budget_buddy_user`

**Schema Routing:**
- `APP_MODE=dev/prod` → `public`
- `APP_MODE=demo` → `demo_{DEMO_PERSONA}`
- `APP_MODE=test` → `test`

**Scripts:** `./scripts/db_start.sh`, `./scripts/db_stop.sh`, `./scripts/db_reset.sh`
**pgAdmin:** http://localhost:5050

**See:** [docs/POSTGRESQL_SETUP.md](docs/POSTGRESQL_SETUP.md) for complete guide

---

## 🗄️ Database Configuration (HISTORICAL - Pre-PostgreSQL)

SQLite was used before January 2026. The verification protocol, common mistakes, and migration steps are archived at [docs/claude/historical-sqlite.md](docs/claude/historical-sqlite.md).

---

## 🔗 Account Linkage (CRITICAL)

**Symptom**: Transactions show bank name but "N/A" for account — `account_db_id` FK not populated.
**Fix**: `python3 scripts/fix_account_linkage.py`
**When to run**: After importing old data, restoring from backup, or seeing "N/A" in account column.
**New imports**: Auto-linked via `save_transactions()` in `database_service.py` (commit fe3c165+).

**See:** [docs/claude/account-linkage.md](docs/claude/account-linkage.md) for verification SQL, edge cases, and full details.

---

## 🕐 Timezone Handling (CRITICAL)

**THE PROBLEM**: SQLite timestamps without 'Z' suffix cause JavaScript to interpret as local time instead of UTC.

**THE SOLUTION**: Always append 'Z' when returning timestamps:
```python
def format_utc_timestamp(timestamp_str):
    if not timestamp_str:
        return None
    dt = datetime.strptime(timestamp_str.split('.')[0], '%Y-%m-%d %H:%M:%S')
    return dt.isoformat() + 'Z'  # Returns: "2026-01-21T16:00:00Z"
```

**Rules**:
- Backend — Store: `datetime.utcnow()` | Return: always use `format_utc_timestamp()` helper
- Frontend — Parse: `new Date("2026-01-21T16:00:00Z")` | Display: `date-fns` formatters

**Common Fields**: `created_at`, `updated_at`, `last_login`, `last_activity_at`, `expires_at`, `deleted_at`

**Test Symptom**: Times off by exact timezone offset (e.g., 5 hours for EST)

---

## 👤 User ID Convention (CRITICAL)

**Standard**: Always get `user_id` from auth context. **NEVER hardcode "default_user"**.

**CORRECT Pattern for API Endpoints**:
```python
from fastapi import Request
from backend.api.middleware.auth import get_current_user_id

async def my_endpoint(request: Request, ...):
    user_id = get_current_user_id(request)  # Always use this!
```

**WRONG Pattern** (DO NOT USE):
```python
user_id = "default_user"  # ❌ NEVER hardcode this!
```

**Symptom of Mismatch**: Data exists but queries return empty (wrong user_id being used).

**Three bug classes covered in depth**: Plaid imports, ORM serialization methods, and CREATE endpoints.
**See:** [docs/claude/user-id-patterns.md](docs/claude/user-id-patterns.md)

---

## 🔐 Authentication System (CRITICAL)

**Requires BOTH**: Middleware registration + Router registration in `/backend/api/main.py`.

**Setup Checklist**:
1. Register `AuthenticationMiddleware` (after CORS, before routers)
2. Register auth router (first router)
3. Install: `pip install email-validator google-auth-oauthlib`
4. Database: Ensure users table + encrypted columns exist

**Common Errors**:
- 401 on all endpoints → Middleware not registered
- 404 on `/api/v2/auth/*` → Router not registered
- `ImportError` → Missing dependencies
- `'State' has no 'user_id'` → Middleware not running

**Endpoint Pattern**:
```python
from backend.api.middleware.auth import get_current_user_id
user_id = get_current_user_id(request)  # Never hardcode!
```

**Test After Changes**: Backend starts clean, dev-login returns 200, protected endpoints work, frontend loads.

---

## ⚠️ Service Method Return Value Contracts (CRITICAL)

**Common Error**: Assuming service returns partial data (e.g., just list) when it returns dict with nested data.

**Always Before Using Service Method**:
1. Read service file's return statement
2. Check actual structure returned
3. Access nested data correctly: `result["categories"]` not `result` directly

**Common Patterns**: Dict with nested data | List of items | Single item or None

---

## 🚨 Startup & Server Configuration (CRITICAL)

**Always Use Scripts**: `./restart_dev.sh` or `./restart_prod.sh`

**If Manual Start Required**:
- ✅ CORRECT: `python -m uvicorn backend.server:app --reload --port 8000`
- ❌ WRONG: `backend.api.main:app` (missing `/api` prefix = 404s)

**CORS Required** (`/backend/api/main.py`):
```python
app.add_middleware(CORSMiddleware, allow_origins=["http://localhost:3000"], allow_credentials=True)
```

### Authentication & Cookie Configuration (CRITICAL)

**Problem**: Cross-origin cookies blocked. **Solution**: Use `setupProxy.js` (NOT package.json proxy).

**Frontend Setup**:
- `frontend/src/setupProxy.js` - Proxy `/api` to `http://127.0.0.1:8000` with `ws: false`
- `frontend/src/config/apiConfig.js` - Use relative path `/api/v2` in dev
- `frontend/src/services/apiClient.js` - MUST use relative URLs in dev (NOT `http://127.0.0.1:8000`)

**Critical Rules**:
1. ✅ Always use `getApiUrl()` helper
2. ✅ NO trailing slashes (causes 307 redirect → bypasses proxy → 401)
3. ✅ Always include `credentials: 'include'` in fetch
4. ❌ NEVER hardcode `http://127.0.0.1:8000` URLs

**Verification**:
- Network tab: Request URL = `localhost:3000/api/v2/*`, `sec-fetch-site: same-origin`, Cookie present
- After changing `apiClient.js`: **Must restart frontend completely**

**Cookie Settings**: `httponly=True, secure=False, samesite="lax"` (omit domain)

---

## 🚀 Production Deployment (Supabase + Railway) (CRITICAL)

**The Rule**: Always use Supabase Transaction Mode (port 6543) + NullPool + preDeployCommand migrations.

**Quick Fix Checklist**:
- ✅ DATABASE_URL uses port 6543 (NOT 5432)
- ✅ NullPool configured (let Supavisor handle pooling)
- ✅ railway.toml has preDeployCommand for Alembic
- ✅ Migrations use IF NOT EXISTS (idempotent)
- ✅ PostgreSQL boolean syntax (`is_deleted = false` not `= 0`)

**Common Error**: `MaxClientsInSessionMode` → Wrong port (using 5432 session mode instead of 6543 transaction mode).

**Fix**: Update DATABASE_URL to port 6543, use NullPool. See `/docs/claude/supabase-railway-deployment.md` for complete guide.

**Configuration** (backend/database/session.py):
```python
engine = create_engine(
    DATABASE_URL,
    poolclass=NullPool,  # Let Supavisor handle pooling
    connect_args={"prepare_threshold": None},  # Disable prepared statements
    execution_options={"isolation_level": "AUTOCOMMIT"}
)
```

---

## Database Service Notes

**Manual Transactions**: Parameter name is `transaction_date` (NOT `date`) in `create_manual_transaction()`. Mismatch causes 500 error.

**Similarity Matching**: `get_similar_unclassified_transactions()` uses fuzzy matching (0.85 threshold) on descriptions + exact merchant name matching. Returns only unclassified transactions (`bb_category_manual = False`).

---

## 🗃️ Transaction Field Name Contract (CRITICAL)

**Problem**: `transform_transaction()` serializes date as `"date"`; DB model uses
`"transaction_date"`. Mismatches cause silent filter no-ops (e.g., `new Date(undefined)`
comparisons always return false).

**Fix applied**: `transform_transaction()` now emits both fields. `useTransactionState.normalizeTransaction()` guarantees `transaction_date` is always set on every transaction in state.

**Rule**: All new code must use `tx.transaction_date`. Never use `tx.date` alone.

**See:** [docs/claude/transaction-field-names.md](docs/claude/transaction-field-names.md)

---

## Data Format Conventions

**Two Formats**:
1. **Monthly (Canonical)**: `spending_by_category` (LIST) with `actual` field - Required by all Buddy AI endpoints
2. **Weekly (Legacy)**: `category_summary` (DICT) with `spent` field - Frontend converts using helpers

**Frontend Helper**: `getNormalizedSpendingData(summary)` from `dataFormatHelpers.js` - Returns LIST format from either source.

**Validation**: Pydantic models (`buddy.py`) enforce strict validation. Sending DICT to planning endpoints = 422 error.

**Smart Batch Update**: After manual classification, shows modal to apply same classification to similar unclassified transactions.

---

## Common Issues & Solutions

**401 Unauthorized**: Frontend bypassing proxy. Check: setupProxy.js exists, NO trailing slashes in endpoints, NO hardcoded URLs (`127.0.0.1:8000`), `credentials: 'include'` in fetch. **Must restart frontend after changes**.

**Failed to fetch**: Check CORS in `main.py`, backend on port 8000, `base_path = "/api/v2"`.

**ModuleNotFoundError**: Run uvicorn from project root, not `/backend` directory.

**Modal not showing**: Transactions already classified OR no similar unclassified transactions found.

**WebSocket errors**: Ensure `ws: false` in setupProxy.js.

**API Rules**: Relative paths in dev, use `getApiUrl()`, NO trailing slashes, include credentials, NEVER hardcode URLs.

---

## Weekly Spending Tracking

**Not weekly budgets** - All budgets monthly. Weekly targets = `(remaining budget) / (weeks remaining)`.

**Adjusted Targets**: Accounts for expected large expenses (rent, insurance). Shows "Target: $X base ($Y with bills)".

**Cross-Month Weeks**: Assigned to month with most days (4+). Transactions counted in actual month. UI shows "Spans Months" badge.

**Key Files**: `database_service.py` (calculate_adjusted_weekly_target), `weekly_check_in.py`, `MonthlyOverviewSection.js`.

---

## Demo Mode System

**Purpose**: Demonstrate with synthetic data. Separate database per persona: `budget_buddy_demo_{persona}.db`.

**Start**: `./restart_demo.sh [young_professional|family|freelancer|recent_grad]`

**Personas**: Young pro ($5.5k/mo, 35% over), Family ($8k/mo, 40% over), Freelancer (variable income, 25% over), Recent grad ($3.5k/mo, 50% over).

**Data Generation**: `python -m backend.scripts.generate_demo_data --persona=PERSONA --months=6`
- Realistic patterns: income, recurring, variable spending, refunds, P2P, duplicates
- Time-based: weekend/weekday/seasonal multipliers
- Persona templates: `/backend/data/persona_templates/*.json`

**Frontend**: Purple banner shows active persona (`ModeIndicator.js`), hidden in prod.

**Safety**: Startup validation prevents mode/DB mismatches. Separate log files.

**Key Files**: `session.py` (routing), `generate_demo_data.py` (CLI), `persona_templates/` (configs).

---

## 📊 Dashboard System

**9 Sections**: Network banner, Welcome header, Quick wins, Daily insight, Monthly overview, Action items, Month projection, Recent transactions/Goals, Sinking funds.

**Architecture**:
- **Data**: `useDashboardData.js` - `Promise.allSettled` fetches 9 endpoints in parallel (failures don't block)
- **Highlights**: `useAdaptiveDashboard.js` - Soft glows for attention (budget variance, pending reviews, unclassified transactions), dismissible 24h
- **Errors**: `SectionError`, `EmptyState`, `NetworkStatusBanner`, `ErrorBoundary` components. Section-specific messages from `errorUtils.js`
- **Mobile**: Bootstrap breakpoints, touch targets ≥44px, reduced animations, `mobile-optimizations.css`

**Key Components**:
- `MonthlyOverviewSection` - Week + budget metrics, cross-month handling, adjusted targets
- `ActionItemsPanel` - Priority-based (critical/high/medium/low), dismissible
- `RecentTransactionsSection` - Table (desktop) / cards (mobile), unclassified banner if ≥5
- `GoalsProgressSection` / `SinkingFundsSection` - Progress bars, status badges, sorted by priority

**Files**: `/components/Dashboard/*`, `/hooks/useDashboardData.js`, `/hooks/useAdaptiveDashboard.js`, `/utils/errorUtils.js`, `/styles/mobile-optimizations.css`

**Testing**: See `/frontend/MOBILE_TESTING_GUIDE.md`. Target Lighthouse ≥90.

---

## 🧪 Testing Infrastructure (2026)

**Overview**: Budget Buddy uses the Testing Trophy model (80% integration, 15% unit, 5% E2E) with modern tooling.

**Full Guide**: See `/TESTING_GUIDE.md` for comprehensive documentation.
**Patterns & workarounds**: See [docs/claude/testing-reference.md](docs/claude/testing-reference.md).
**Jest mocking patterns**: See [docs/claude/jest-patterns.md](docs/claude/jest-patterns.md).

### Tech Stack

**Frontend**: Vitest 2.1.8+, React Testing Library 16.2.0, MSW 2.7.0, Playwright 1.49.1
**Backend**: pytest 7.0.0+ (randomized), Pydantic Settings v2, testcontainers-python + PostgreSQL 16, pytest-cov 4.1.0+
**CI/CD**: Coverage ratcheting via Codecov, 100% patch coverage, Node 18/20/22 + Python 3.11/3.12 matrix

**📚 Detailed Guides**:
- **[Testing Recommendations](/docs/TESTING_RECOMMENDATIONS.md)** - ⭐ START HERE
- **[Testing Best Practices](/docs/TESTING_BEST_PRACTICES.md)**
- **[Fixture Architecture](/docs/FIXTURE_ARCHITECTURE.md)**
- **[Configuration Guide](/docs/CONFIGURATION_GUIDE.md)**
- **[Local CI Testing](/docs/LOCAL_CI_TESTING.md)**

### Quick Commands

```bash
# Frontend (Vitest)
cd frontend
npm test                    # Watch mode
npm run test:ci             # CI mode (run once)
npm run test:coverage       # With coverage report

# Backend (pytest)
cd backend
pytest                                    # All tests (randomized order)
pytest --cov=backend                      # With coverage
pytest --randomly-seed=12345              # Reproduce specific test order
pytest backend/tests/integration/         # Integration only (requires Docker)
CI=true pytest backend/tests/             # Force PostgreSQL (CI environment)

# E2E (Playwright)
npx playwright test                      # All browsers
npx playwright test --project=chromium   # Specific browser
npx playwright test --ui                 # Interactive mode
```

**Test Suite**: ~430 tests (86 frontend integration, 314 backend, 30 E2E)

**⚠️ CRITICAL**: See `/docs/TESTING_PRACTICES.md` before writing backend integration tests (SQLAlchemy gotchas, circular dependency cleanup).

---

## 🚀 CI/CD Best Practices (CRITICAL)

**Key Rules**:
- After adding/updating npm packages: `rm -rf node_modules package-lock.json && npm install` then re-verify build
- Main branch is protected — all checks must pass before merge (lint, tests on 3.11/20.x/22.x, e2e chromium)
- If merge blocked: check GitHub Actions, fix failing check, push, wait for re-run

**See:** [docs/claude/cicd-practices.md](docs/claude/cicd-practices.md) for package management details, CI protocol, and branch protection configuration.

---

## 🤖 ML Model Training & Fallback (CRITICAL)

**Problem**: Training fails in production when database has 0 manually classified transactions.
**Solution**: Graceful fallback to `backend/ml/data/labeled_transactions.json` (1,079 transactions).
**Verify**: `python -m backend.scripts.verify_training_data`
**Fallback logic**: `backend/ml/data/training_utils.py::load_labeled_transactions()`

**See:** [docs/claude/ml-training-fallback.md](docs/claude/ml-training-fallback.md) for flow diagram, logging indicators, and full file list.

---

**Last Updated**: February 3, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fedickinson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fedickinson)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
