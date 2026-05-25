## hodlo-solona-hackathon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Automated meme coin trading bot. Ingests real-time token signals from community WebSocket feeds, filters them through a multi-dimensional pipeline, executes trades on-chain (Solana, BSC, Ethereum, Base), and monitors positions for TP/SL/timeout/rug exits. React dashboard for monitoring and config.

## Commands

**Backend:**
```bash
pip install -r requirements.txt
cd backend && uvicorn main:app --reload --port 9000
```

**Frontend:**
```bash
cd frontend && npm install && npm run dev
```

**Start both (Windows):**
```bash
start.bat
```

- Frontend: http://localhost:5173
- API docs: http://localhost:8000/docs

## Architecture

```
Community WebSocket → ca_listener → trade_engine → ave_client → position_monitor
                                                                      ↓
                                              broadcaster → Frontend WebSocket
```

All backend services are started in `backend/main.py` lifespan and run as async background tasks.

**Key services** (`backend/services/`):
- `ca_listener.py` — WebSocket client receiving token CA signals with sender reputation and market metadata
- `trade_engine.py` — Filtering pipeline: sender reputation → anti-FOMO → community heat → market data → safety checks (honeypot, mint authority, holder concentration)
- `ave_client.py` — On-chain execution via AVE Trading API across 4 chains; handles nonce management, receipt polling, Token-2022/pump.fun
- `position_monitor.py` — 10s polling loop; exits on TP/SL/timeout/rug (on-chain balance = 0)
- `broadcaster.py` — WebSocket push to frontend for real-time updates
- `wallet_manager.py` — AES-encrypted wallet storage

**Database** (`backend/database.py`, SQLite async via aiosqlite):
- `ConfigModel` — key-value config (all thresholds tunable at runtime via admin panel, no restart needed)
- `Position` — open holdings
- `Trade` — closed trades with PnL, gas fees, TX hashes
- `CaFeed` — permanent signal log (all received signals + outcomes, intended for ML training)
- `TokenDetail` — cached AVE token metadata

**Routers** (`backend/routers/`): 40+ REST endpoints for positions, trades, wallet, analytics, config, admin, AI chat, and WebSocket.

## Environment

Copy `.env.example` to `.env`. Required vars:
- `AVE_API_KEY` — trading API key
- `CA_WS_URL` — community signal WebSocket (e.g. `ws://host:3000/token`)
- `WALLET_MASTER_PASSWORD` — AES encryption key for wallets
- `JWT_SECRET` — JWT signing key

Optional: `BACKEND_PORT` (default 9000), `ADMIN_PASSWORD`, `DEMO_WALLET_MNEMONIC`.

---
> Source: [SmallICeplay/hodlo-solona-hackathon](https://github.com/SmallICeplay/hodlo-solona-hackathon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
