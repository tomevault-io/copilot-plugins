## to-the-moon

> **To The Moon** is an advanced Solana Token Scoring System built to automatically track, analyze, and score tokens migrated from Pump.fun to Solana DEXs. It employs a sophisticated "Hybrid Momentum Model" to evaluate token arbitrage potential through four core metrics: Transaction Acceleration, Volume Momentum, Token Freshness, and Orderflow Imbalance, all stabilized by EWMA smoothing.

# To The Moon 🚀 - Project Context

## Project Overview

**To The Moon** is an advanced Solana Token Scoring System built to automatically track, analyze, and score tokens migrated from Pump.fun to Solana DEXs. It employs a sophisticated "Hybrid Momentum Model" to evaluate token arbitrage potential through four core metrics: Transaction Acceleration, Volume Momentum, Token Freshness, and Orderflow Imbalance, all stabilized by EWMA smoothing.

### Architecture & Technology Stack

The project uses a Domain-Driven Design (DDD) architecture and is split into a Python backend and a React frontend.

**Backend:**
- **Language/Framework:** Python 3.10+, FastAPI
- **Database/ORM:** PostgreSQL 14+, SQLAlchemy 2.x, Alembic
- **Core Libraries:** Pydantic v2 (validation/settings), APScheduler (background tasks), WebSockets, HTTPX
- **Integrations:** DexScreener API, Pump.fun WebSocket

**Frontend:**
- **Framework:** React 18 with TypeScript
- **Tooling:** Vite, Tailwind CSS (via PostCSS)
- **State/Data Fetching:** `@tanstack/react-query`, `react-router-dom`
- **UI Components:** Radix UI, Recharts, Lucide React, `react-hook-form`

---

## Building and Running

### Prerequisites
- **Python 3.10+**
- **Node.js 18+** & npm
- PostgreSQL 14+

### Backend Setup
1. **Install dependencies:**
   ```bash
   python3 -m pip install -r requirements.txt
   ```
2. **Environment Configuration:**
   ```bash
   cp .env.example .env
   # Update .env with appropriate settings (e.g., DATABASE_URL, APP_ENV)
   ```
3. **Initialize Database Migrations:**
   ```bash
   python3 -m alembic upgrade head
   ```
4. **Start Development Server:**
   ```bash
   make run
   # This runs: PYTHONPATH=. uvicorn src.app.main:app --host 0.0.0.0 --port 8000 --reload
   ```

### Frontend Setup
1. **Install dependencies and run the dev server:**
   ```bash
   cd frontend
   npm install
   npm run dev
   ```
2. **Build for production:**
   ```bash
   cd frontend && npm run build
   ```

---

## Development Conventions & Commands

A `Makefile` is provided in the root directory to streamline common development tasks.

- **Formatting (Python):** Uses `black`.
  ```bash
  make format
  ```
- **Linting (Python):** Uses `ruff`.
  ```bash
  make lint
  ```
- **Testing:** Uses `pytest` (configured in `pytest.ini` for `asyncio` and `tests/` directory).
  ```bash
  make test
  ```
- **Database Operations:**
  - Generate migration: `python3 -m alembic revision -m "description" --autogenerate`
  - Apply migrations: `python3 -m alembic upgrade head`
  - Populate test data: `PYTHONPATH=. python3 scripts/smoke_db.py`

### Guidelines
- **Code Style:** Strict adherence to Python formatting (`black`) and linting (`ruff`). Frontend adheres to standard ESLint/Prettier patterns (if configured).
- **Architecture Integrity:** Maintain the clear separation established by the Domain-driven design (e.g., `src/domain`, `src/adapters`, `src/core`, `src/app`).
- **Resilience:** The backend features autonomous self-healing, circuit breakers, smart scheduling, and fallback mechanisms. Ensure any new integrations or services plug into this resilience framework properly.
- **Database:** Ensure all schema changes are tracked via Alembic migrations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/super-sh1z01d)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/super-sh1z01d)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
