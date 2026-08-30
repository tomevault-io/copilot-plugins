## sortino

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev              # Start both frontend (port 3000) and backend (port 3001) concurrently
npm run dev:frontend     # Vite dev server only
npm run dev:backend      # Express server only
npm run build            # Production build (outputs to /dist)
npm start                # Production mode (NODE_ENV=production)
npm start:win            # Production mode, Windows
```

No test or lint scripts are configured.

## Architecture

**Sortino** is a full-stack algorithmic trading platform. It connects to brokerage accounts via the Alpaca API and executes trades based on ML model predictions for DOW-30 stocks.

### Stack

- **Frontend**: React 19 + TypeScript, Vite (dev server port 3000), React Router 7 (HashRouter), Recharts, Tailwind CSS via CDN
- **Backend**: Express 5 (port 3001), ES modules (`type: "module"` in package.json)
- **Database**: PostgreSQL via Neon (`lib/db.js` — connection pooling, SSL)
- **Proxy**: Vite proxies `/api/*` → `http://localhost:3001` in dev

### Frontend Structure

`index.html` → `index.tsx` → `App.tsx` (HashRouter)

Five main pages: Dashboard, Paper Trading, Live Trading, History, Settings. Real-time data uses 30-second polling intervals.

### Backend Structure

`server.js` is the entry point. It mounts 11 API route handlers from the `/api` directory using a Vercel-style handler pattern (`async function handler(req, res)`). The same file handles GET/POST/PATCH/DELETE by switching on `req.method`.

**API surface:**
```
/api/accounts           – CRUD for trading accounts (encrypts API keys on write)
/api/bot-status         – Bot running state, strategy assignment, per-account settings
/api/trading            – Bot control and health-check trigger for cron
/api/trades             – Trade history
/api/stats              – Aggregate trading statistics
/api/market-prices      – Current prices for DOW-30 tickers
/api/account-portfolio  – Positions and equity for an account
/api/dashboard-summary  – Combined Paper + Live portfolio view
```

### Trading Loop

`api/trading/loop.js` is the core execution engine. On each heartbeat:

1. Fetches ML model predictions for DOW-30 from `MODEL_API_URL` (env var, defaults to `localhost:5000`)
2. Applies a rolling window of 5 predictions per ticker to smooth signals
3. Applies a confidence threshold (0.45) and dead zone (0.30–0.45)
4. Executes buy/sell orders via Alpaca broker API
5. Persists trades to PostgreSQL

The loop is market-aware — only trades 9:30 AM – 4:00 PM ET on weekdays.

### Credential Security

All Alpaca API keys/secrets are AES-256-GCM encrypted before being stored in the database (`lib/encryption.js`, uses `ENCRYPTION_KEY` env var). The frontend always receives masked credentials (`***`). Decryption only happens server-side inside the trading loop. The `?decrypt=true` query param requires localhost origin or a token.

### Database Tables

- `accounts` — trading accounts with encrypted credentials
- `trades` — executed trades with PnL, strategy, ticker, action
- `bot_state` — running/always_on flags per account
- `model_versions` — active model metadata (strategy, version)
- Portfolio history tables (for charting)

### Environment Variables

```
DATABASE_URL      # Neon PostgreSQL connection string
ENCRYPTION_KEY    # Hex string for AES-256-GCM credential encryption
MODEL_API_URL     # URL for ML model prediction server
```

### Agent Logging

Some files (`api/trading/index.js`, `api/trading/loop.js`) contain debug instrumentation that POSTs JSON to `http://127.0.0.1:7246/ingest/...`. These are wrapped in `// #region agent log` / `// #endregion` comments and silently swallow errors. This is intentional development tooling — do not remove.

---
> Source: [danielpj17/Sortino](https://github.com/danielpj17/Sortino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
