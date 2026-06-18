## neontrade-ai

> FastAPI backend + React Native / Expo Web frontend. Single-user app (Sergio Castellanos). Capital ~190.88 USD, MANUAL mode. GitHub webhook auto-deploys `main` to EasyPanel.

# Atlas — Automated trading on Capital.com, TradingLab strategies

FastAPI backend + React Native / Expo Web frontend. Single-user app (Sergio Castellanos). Capital ~190.88 USD, MANUAL mode. GitHub webhook auto-deploys `main` to EasyPanel.

## Quick start

```bash
# Tests (suite ~1391 tests in 73-96s)
pytest backend/                        # full suite
pytest backend/test_simulation_end2end.py -v
pytest backend/ --cov=backend -q       # with coverage (~69% baseline)

# Run backend locally
python3 -m uvicorn backend.main:app --port 8000 --reload
# (Python deps installed at system level; `uvicorn` CLI may not be on PATH — use `-m`.)

# Frontend dev
cd frontend && npx expo start --web

# Deploy (auto via webhook)
git push origin main                   # triggers EasyPanel rebuild of project=n8n service=neontrade_ai
```

## Pytest conventions

- `rootdir = backend/` (per `backend/pytest.ini`). Tests import with `from config import settings`, **not** `from backend.config`.
- `asyncio_mode=auto` — async tests don't need `@pytest.mark.asyncio`.
- 5 legacy files skipped via `collect_ignore` in `backend/conftest.py` (they `sys.exit()` at module scope).

## Repo layout

- `backend/` — FastAPI app, entrypoint `main.py`, config in `config.py` (Pydantic settings)
  - `core/` — trading_engine, position_manager, risk_manager, market_analyzer, news_filter, security, screenshot_generator
  - `strategies/base.py` — BlueStrategy, RedStrategy, GreenStrategy, WhiteStrategy, BlackStrategy, PinkStrategy
  - `broker/capital_client.py` — Capital.com REST client; `ibkr_client.py` alternate
  - `notifications/` — Gmail OAuth2, alerts
  - `data/*.json` — runtime persistence (session cache, news cache, risk overrides) — gitignored
  - `logs/` — loguru rotated daily in UTC, also mounted from `atlas-logs` volume in prod
- `frontend/` — Expo Router + React Navigation v7 (web build deployed)
  - `App.tsx` — tab nav with `FocusOnlyMount` HOC (desktop unmounts inactive tabs)
  - `src/screens/` — Home, Trade, Market, Log, Settings + sub-tab screens
- `docs/superpowers/` — specs, plans, audit artifacts (kept in git for audit trail)
- `docs/mentoria/` — final deliverable docs for TradingLab mentorship

## Trading rules (post-audit 2026-04-17, commit `73e14db`)

Canonical source: Trading Plan PDF (Alex Ruiz). Oral instructions in `TradingLab_Notas/` are a fallback but the PDF wins on conflict.

- `be_trigger_method="pct_to_tp1"` (0.50 half-way to TP1). The old `"risk_distance"` only works at R:R 2:1.
- `position_management_style="cp"` — Alex CP (short-term) with EMA 5 (principal trail) + EMA 2 (emergency) in M5. **Not EMA 50.**
- Trailing activates only **after** swing high/low (of last pre-entry pullback) is broken — see `_handle_be_phase` swing gate.
- BLUE: `tp1 = swing anterior`, `tp_max = EMA 4H 50` (B/C) or Fib 1.272/1.618 (A). Do not invert.
- RED Wave 3 + HTF favor: `tp_max = Fib 1.0` default (1.272/1.618 are escalation only when Fib 1.0 reached + momentum confirms).
- `_check_weekly_ema8_filter` applies **only to crypto instruments** (guard with `_is_crypto_instrument`). Applying it to forex/indices blocks BLACK + valid setups.
- `risk_day_trading=0.01`, `risk_scalping=0.005`, `risk_swing=0.01`, `max_total_risk=0.05` are **intentional caps for capital < $500**. PDF says 3% swing / 7% total. Comments should reflect this — never claim "NON-NEGOTIABLE per mentorship" (it's not).
- `scalping_enabled=False` until day trading is mastered.
- `trading_hours_utc="07:00-21:00"`, `close_before_friday_hour=20`, `no_new_trades_friday_hour=18`.
- `avoid_news_minutes_after=30` (covers NFP/CPI/FOMC volatility window).

## Gotchas

### Backend
- `risk_manager.calculate_position_size()` returns **signed units** (negative for SELL). Don't `if units <= 0: reject` — that rejects all SELL. Check `units == 0` for "no trade" (insufficient margin).
- `PositionPhase` and `ManagedPosition` are imported at module level in `core/trading_engine.py:26`. Never add local `from core.position_manager import ...` inside loops — it shadows the module binding and breaks other branches.
- Capital.com session token cached in `/app/data/capital_session.json` (TTL 540s). Every `401` invalidates the cache. Don't clear unless debugging auth. Keep `data/*.json` in `.gitignore`.
- `_get()` / `_post()` / `_put()` / `_delete()` in `broker/capital_client.py` short-circuit on 400/401/403/404/422 (via `_is_permanent_error`). Don't retry them.
- Security (`core/security.py`): when `auth_enabled=True` and `api_keys={}`, `validate_key` returns `False` (fail-closed). Generate key via CLI before first startup, or disable `auth_enabled` in dev.
- Websocket `ConnectionManager` uses `asyncio.Lock` — both `connect` and `disconnect` are async.
- `api/routes.py` uses `datetime.now(timezone.utc)` (UTC always). Loguru also rotates in UTC.
- `screenshot_generator.capture_trade_open/close` wrap matplotlib calls in `run_in_executor` — don't call `_generate_candlestick_chart` from the event loop directly.
- SMT divergence logs at **DEBUG**, not WARNING. That fixed 95.8% of log noise. Don't revert.
- News calendar falls back to `/app/data/news_cache.json` (<48h) when FairEconomy + TradingEconomics both fail. Don't disable without alternative.

### Frontend
- React Navigation v7 removed `unmountOnBlur`. Use the `FocusOnlyMount` HOC in `App.tsx` (already wrapped). Desktop was mounting all 5 tabs simultaneously → 5× polling; mobile was fine.
- Sub-tab switcher lives in `App.tsx` (`SubTabScreen`). Screens must NOT re-render their own `SubNavPills` — that was the duplicate-bar bug.
- API key: `window.__ATLAS_API_KEY__ = atob("<base64>")` injected at build into HTML. Send as `X-API-Key` header. Decode with `echo <base64> | base64 -d`.
- Only 1 `role="tablist"` per screen. If you see 2, something regressed.

## Infra / ops

- **Prod URL:** `https://n8n-neontrade-ai.zb12wf.easypanel.host/`
- **EasyPanel:** `https://zb12wf.easypanel.host` (login: `scastellanos@phinodia.com`; password in personal password manager — DO NOT commit)
  - project `n8n` / service `neontrade_ai`
  - tRPC auth: `POST /api/trpc/auth.login?batch=1` body `{"0":{"json":{"email":"...","password":"..."}}}` → Bearer token
  - logs: `/ws/serviceLogs` (container live buffer ~30 min) or `/ws/containerShell` + `base64` for rotated `/app/logs/atlas_YYYY-MM-DD.log`
- **Volumes:** `atlas-data` (`/app/data`) and `atlas-logs` (`/app/logs`). Session cache and news cache persist here across redeploys.
- **Broker:** Capital.com REST, account `314623104804541636`, MANUAL mode. Balance ~$190.88 USD.
- **Apalancamientos activos:** forex/indices/commodities 100:1, stocks/crypto 20:1, bonds/rates 200:1.

## Testing patterns

- Bypass heavyweight `TradingEngine.__init__` for unit tests with `engine = TradingEngine.__new__(TradingEngine); engine.broker = MagicMock(); ...` — see `test_simulation_bugs_fix.py`.
- Full E2E fixture with patched `_create_broker` etc. — see `test_simulation_end2end.py::_make_engine`.
- Use `AsyncMock` for async broker methods; ensure `await` all mocked coroutines or the warning "coroutine was never awaited" surfaces.

## Audit trail

Recent audit 2026-04-17 (commit `f34ef12` deployed, verified `73e14db`): 4 Opus 4.7 subagents ran in parallel worktrees over mentorship ↔ code, EasyPanel logs, Playwright screenshots, and static analysis. 38 findings consolidated in `docs/superpowers/audits/2026-04-17-findings-consolidated.md`. 20 fixes deployed in 31 commits. Suite: 1391 passed / 0 failed.

Final deliverable for mentorship (3 real trades with screenshots + analysis) pending Mon 2026-04-20 market open. Scaffold at `docs/mentoria/2026-04-17-entregable-3-trades.md`.

---
> Source: [Seryi358/neontrade-ai](https://github.com/Seryi358/neontrade-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
