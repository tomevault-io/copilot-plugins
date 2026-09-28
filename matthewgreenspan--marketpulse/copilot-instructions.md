## marketpulse

> A full-stack real-time market dashboard. Backend (FastAPI + PostgreSQL) + Frontend (TypeScript, no framework, Chart.js). Single-origin deploy — FastAPI serves both the API and the static frontend.

# MarketPulse

## What It Is
A full-stack real-time market dashboard. Backend (FastAPI + PostgreSQL) + Frontend (TypeScript, no framework, Chart.js). Single-origin deploy — FastAPI serves both the API and the static frontend.

## Problem Statement (in progress)
Retail traders have market data but no systematic way to identify entry points or measure whether their decisions are working. MarketPulse aims to fix this by combining live market data with signal generation and trade performance tracking.

## Tech Stack
- **Backend:** Python, FastAPI, PostgreSQL, SQLAlchemy ORM, APScheduler, slowapi
- **Auth:** JWT (python-jose) + bcrypt
- **Data sources:** CoinGecko (crypto, batched), yfinance (stocks, per-symbol)
- **Frontend:** TypeScript (compiled to ES modules via tsc), Chart.js, plain CSS Grid — no framework
- **Testing:** pytest
- **GitHub:** https://github.com/MatthewGreenspan/MarketPulse

## What's Built

### Backend
- RESTful API with 4 routers: assets, auth, watchlist, alerts
- 5-table schema: Asset, PriceHistory, Watchlist, Alert, User
- JWT auth + bcrypt password hashing; same 401 for bad email/password (no email enumeration)
- Per-IP rate limiting via slowapi (60/min global, 5/min on auth routes)
- APScheduler: crypto every 5 min (CoinGecko batch), stocks every 5 min (yfinance per-symbol)
- Alert checker runs after each fetch — flips is_triggered/triggered_at when price crosses threshold
- Guest/authenticated dual-access; all watchlist + alert queries scoped by user_id (IDOR prevention)
- Email validation on signup (format + disposable domain blocklist)
- Public GET /assets/summary — latest price, 24h % change, volume per asset

### Frontend
- TypeScript compiled to ES modules (no framework, direct DOM)
- Auth gate → dedicated login/signup page (not a modal)
- Dashboard: stat cards (top gainer/loser, most active, assets tracked), Chart.js price chart, filterable/sortable Assets panel, watchlist + alerts row
- Guest mode — "Continue as guest" shows real data; watchlist/alerts show locked preview prompting signup
- Dark/light theme via CSS custom properties
- Mobile responsive
- Auth token stored in memory (not localStorage — XSS safe, but a refresh logs you out)
- Inline field errors on auth forms, toast notifications elsewhere

## Known Weak Spots (from ARCHITECTURE.md)
- Inconsistent error handling: watchlist/alerts return 200 + {"error"} instead of raising HTTPException
- N+1 query in GET /watchlist/ (alerts.py's with_assets shows the batched fix)
- In-memory token = refresh drops you to sign-in
- Sequential integer IDs (UUIDs would remove enumeration risk)
- No auth-flow tests yet
- datetime.utcnow() deprecated in modern Python (should use datetime.now(timezone.utc))

## What's Left / In Progress
- **Prediction/entry point engine** (7-day moving average, % below recent high, RSI, suggested entry price per asset)
- Trade logging + success rate / P&L tracking
- Email notifications when alerts trigger
- UUID migration on all tables
- CORS restrictions
- Fix token persistence (httpOnly cookie or sessionStorage)
- Email verification on signup
- Password reset flow
- S&P 500 assets (blocked on data source — yfinance free tier is too slow for 500 symbols)
- Data retention policy (auto-delete price_history > 90 days)
- Deployment (Railway backend, Railway/Vercel frontend, GitHub Actions CI/CD)
- Chart time range selector

## Resume Bullets (CURRENT — needs update)
Current bullets are outdated:
- Still mention Alpha Vantage (replaced by yfinance)
- Don't mention the frontend at all
- Don't mention TypeScript, Chart.js, or the guest mode

## Notes
- Mentor feedback: needs a clear problem statement ("what issue does it solve?")
- Adding entry point suggestions + trade tracking = transforms it from a dashboard into a decision-making tool
- ARCHITECTURE.md has "be ready to explain" annotations for every file — good interview prep

---
> Source: [MatthewGreenspan/MarketPulse](https://github.com/MatthewGreenspan/MarketPulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
