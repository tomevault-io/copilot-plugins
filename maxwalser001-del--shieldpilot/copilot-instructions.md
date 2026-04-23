## shieldpilot

> - Max 6-step plan, then execute. No over-planning.

# ShieldPilot - Claude Code Working Agreement

## Non-Negotiable Operating Rules

### 1. Action First
- Max 6-step plan, then execute. No over-planning.
- If the user gives a clear directive, start coding immediately.

### 2. Root Cause Before Patching
- For ANY bug: identify the exact file, function, and line responsible BEFORE editing.
- Explain WHY the current code behaves wrong, not just WHAT to change.
- Never apply the same fix twice. If it failed, switch approach and explain why.

### 3. Verification Is Mandatory
- Run `python3 -m pytest tests/ -x -q --tb=short` after every meaningful change.
- A task is NOT done until tests pass. No exceptions.
- If tests don't cover the change, state that explicitly.

### 4. Scope Control
- Handle fixes ONE AT A TIME, each with a clear acceptance criterion.
- Format: "DONE when: [specific measurable condition]"
- Do not batch multiple unrelated changes into a single edit.

### 5. Tool Switching Compliance
- If the user wants to switch tools or workflows, help them do it.
- Never argue to stay in a tool the user wants to leave.

### 6. Preview Honesty
- If you cannot provide a real preview, say so immediately.
- Offer concrete alternatives: "Start the server with `python3 -m uvicorn sentinelai.api.app:app --port 8420` and open http://localhost:8420"
- Never fake screenshots or use ASCII art as a substitute.

---

## Project-Specific Gotchas

### Python / FastAPI
- Use `python3` not `python` on macOS.
- Use `httpx` not `requests` (requests is not installed).
- Internal package is `sentinelai/` even though branding is "ShieldPilot".
- SQLite does not support `SELECT ... FOR UPDATE` or row-level locking.

### ShieldPilot Hook
- The pre-tool-use hook (`sentinelai.hooks.sentinel_hook`) blocks commands with risk score >= 80.
- `rm -rf`, complex `curl` commands, and destructive filesystem ops WILL be blocked.
- If daily command limit (50 free) is reached, disable billing in `sentinel.yaml` temporarily.
- The hook uses the **Adapter Layer** (`sentinelai/adapters/`) to auto-detect input format:
  - Claude Code: `{"tool_name": "...", "tool_input": {...}}`
  - OpenClaw: `{"event": "preToolExecution", "tool": {"name": "...", "parameters": {...}}}`
  - Generic: `{"command": "...", "tool": "Bash"}`
- Fallback: if adapters fail to import, the hook falls back to direct Claude Code JSON parsing.
- Rate limiter blocks ALL bash for 60s after 5 injection detections in 60s. Each retry resets the timer.

### Database / Migrations
- SQLite with SQLAlchemy ORM at `sentinelai/logger/database.py`.
- Production migration runner: `sentinelai/migrations/runner.py` (backup, integrity check, Alembic, rollback on failure).
- Manual migrations via `ALTER TABLE` in `database.py:migrate_database()`.
- Always verify column exists after migration by reading the inspector.

### Frontend (Vanilla JS SPA)
- No build step. Hash-based routing. No framework.
- `sentinelai/web/static/js/app.js` is the main SPA entry point.
- CSS variables define the dark theme: bg `#0D1117`, accent `#39D2C0`.
- All user-controlled content MUST be escaped via `escapeHtml()` or `_sanitize_text()`.

### Auth
- JWT with HS256, stored in localStorage as `sentinel_token`.
- Super-admin: configured via `SHIELDPILOT_SUPER_ADMIN_EMAIL` / `SHIELDPILOT_SUPER_ADMIN_PASSWORD` env vars. Unlimited tier, bypasses all limits.
- API key auth via `X-API-Key` header (SHA-256 hashed in DB).
- Local-first mode: `config.auth.local_first=True` skips JWT for localhost (127.0.0.1, ::1).
- Google OAuth: callback redirects (302) to `/login?token=<jwt>`.

### Billing
- Tiers: free / pro / enterprise / unlimited (super-admin = unlimited).
- Rate limiter: DB-backed sliding window in `sentinelai/api/routers/_shared.py`.
- Circuit breaker: in-memory, tracks injection attempts per source.
- Stripe integration: checkout, portal, webhooks at `/api/billing/*`.

### Secrets - NEVER commit to repo
- JWT secret key, Google OAuth client secret, Stripe keys, super-admin password.
- Set via environment variables (see sentinel.yaml comments).

---

## Architecture

### Backend (`sentinelai/api/`)
- **Routers:** `sentinelai/api/routers/` — split by domain (auth, commands, incidents, scans, dashboard, settings, billing, admin, export, library, teams, rules, health, activity, legal, config, ratelimit, reports)
- **Services:** `sentinelai/services/` — business logic (auth, billing, user, team, rules, report, library, webhook, rate_limit, tenant)
- **Auth:** `sentinelai/api/auth.py` (JWT), `sentinelai/api/deps.py` (dependency injection, get_current_user)
- **App factory:** `sentinelai/api/app.py` — `create_app()` takes no arguments, config via `deps._config`

### Risk Engine (`sentinelai/engine/`)
- 9 analyzers: destructive_fs, credential_access, network_exfil, obfuscation, persistence, privilege_escalation, supply_chain, malware_patterns, injection
- Scoring: 0-100, thresholds configurable (default: allow < 40, warn 40-79, block >= 80)

### Scanner (`sentinelai/scanner/`)
- 3-pass prompt injection scanner with 178+ patterns in 19 categories
- 13-step sanitizer pipeline
- Circuit breaker for repeated injection attempts

### Adapters (`sentinelai/adapters/`)
- `ClaudeCodeAdapter`, `OpenClawAdapter`, `GenericAdapter`
- `detect_platform(raw)` auto-selects based on JSON shape
- Used by the hook for multi-platform support

### Deployment
- **Docker:** `Dockerfile` (multi-stage, Python 3.12), `docker-compose.yml`
- **CI/CD:** `.github/workflows/ci.yml` (lint, test matrix 3.9-3.12, Docker smoke)
- **Release:** `.github/workflows/release.yml` (tag-triggered, PyPI + GHCR + GitHub Release)
- **Migration runner:** `sentinelai/migrations/runner.py` (backup, integrity, Alembic, rollback)
- **Backup/restore:** `scripts/backup.sh`, `scripts/restore.sh`

---

## Verification Workflow

### Before Reporting Any Task Complete:
```bash
# 1. Run tests (fast, catches regressions)
python3 -m pytest tests/ -x -q --tb=short

# 2. Import check (catches syntax errors across all modules)
python3 -c "from sentinelai.api.app import create_app; print('OK')"

# 3. If frontend was changed, verify templates parse
python3 -c "from pathlib import Path; p=Path('sentinelai/web/templates'); [print(f.name, 'OK') for f in p.glob('*.html')]"
```

### To Start Dev Server:
```bash
python3 -m uvicorn sentinelai.api.app:app --host 0.0.0.0 --port 8420 --reload
```

### URLs to Check Locally:
- Landing page: http://localhost:8420/login
- Dashboard (requires auth): http://localhost:8420/
- API docs: http://localhost:8420/api/docs
- Health check: http://localhost:8420/api/health

---

## CLI Commands
```
sentinel run "command"       # Evaluate + execute a shell command
sentinel scan file.txt       # Scan file for prompt injection
sentinel logs                # Browse activity logs
sentinel status              # System overview and health
sentinel perf                # Performance statistics
sentinel config --validate   # View / validate configuration
sentinel verify              # Verify log chain integrity
sentinel init                # Set up ShieldPilot for a project
sentinel dashboard           # Launch the web dashboard
sentinel hook install        # Install Claude Code hook
sentinel hook uninstall      # Remove Claude Code hook
sentinel hook status         # Show hook installation status
sentinel hook test "cmd"     # Test what the hook would decide
```

---

## File Reference

- **API routers:** `sentinelai/api/routers/*.py` (18 router files)
- **Services:** `sentinelai/services/*.py` (business logic layer)
- **Auth:** `sentinelai/api/auth.py`, `sentinelai/api/deps.py`
- **Database models:** `sentinelai/logger/database.py`
- **Config:** `sentinelai/core/config.py` + `sentinel.yaml`
- **Adapters:** `sentinelai/adapters/` (base, claude_code, openclaw, generic)
- **Risk engine:** `sentinelai/engine/` (9 analyzers + engine.py)
- **Scanner:** `sentinelai/scanner/` (patterns, sanitizer, scanner, circuit_breaker)
- **Hook:** `sentinelai/hooks/sentinel_hook.py`
- **Frontend SPA:** `sentinelai/web/static/js/app.js`, `components.js`
- **CSS:** `sentinelai/web/static/css/sentinel.css`
- **CLI:** `sentinelai/cli/main.py`
- **Migrations:** `sentinelai/migrations/runner.py`
- **Tests:** `tests/` (2261 tests)
- **Coverage report:** `features/COVERAGE-REPORT.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxwalser001-del) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
