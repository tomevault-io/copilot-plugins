## sdx-react-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SDX MEICAN React Application - A React-based network topology visualization and L2VPN connection management system for AtlanticWave-SDX. Replaces the legacy PHP MEICAN system.

## Build & Development Commands

```bash
# Install dependencies (--legacy-peer-deps required for react-leaflet compatibility)
npm install --legacy-peer-deps

# Start frontend dev server (runs on http://127.0.0.1:5173)
npm run dev

# Start backend server (runs on port 3004 by default)
cd backend && npm start
# Or with auto-reload:
cd backend && npm run dev

# Build for production
npm run build

# Lint
npm run lint

# Preview production build
npm run preview
```

**Important**: Both frontend and backend must run simultaneously for full functionality.

## Architecture

### Frontend (React + Vite + TypeScript)

- **Entry Point**: `src/main.tsx` → `src/App.tsx`
- **Routing**: Custom page-based routing in App.tsx (not React Router), pages defined as `Page` type union
- **Path Alias**: `@/` maps to `src/` directory

**Key Directories**:

- `src/components/pages/` - Page components (Dashboard, LandingPage, LoginPage, etc.)
- `src/components/ui/` - shadcn/ui components (Radix UI primitives)
- `src/lib/` - Core services and utilities:
  - `api.ts` - SDX API client (topology, L2VPN CRUD operations)
  - `session.ts` - SessionManager class for auth state (localStorage-based)
  - `token-storage.ts` - OAuth token management
  - `config.ts` - App configuration (OAuth credentials, API endpoints)
  - `topology-processor.ts` - Network topology data transformation

### Backend (Express.js)

Located in `/backend/` with its own package.json.

- **Purpose**: OAuth token exchange proxy, email verification, API proxy to SDX
- **Main File**: `backend/server.js`
- **Endpoints**:
  - `POST /oauth/exchange` - OAuth code-to-token exchange for ORCID/CILogon
  - `GET /api/topology` - Proxies SDX topology API
  - `/api/l2vpn/1.0` - Proxies L2VPN operations (GET, POST, PATCH, DELETE)
  - Email verification endpoints

### Authentication Flow

1. User selects provider (ORCID or CILogon) on LandingPage
2. Redirected to OAuth provider's auth URL
3. Callback pages (`ORCIDCallbackPage`, `CILogonCallbackPage`) handle code exchange via backend
4. Email verification required before dashboard access
5. Tokens stored via `TokenStorage`, session via `SessionManager`

### State Management

- **Session**: `SessionManager` class in `src/lib/session.ts` - localStorage-based, 24hr timeout
- **Tokens**: `TokenStorage` class handles OAuth tokens with expiry checking
- **UI State**: React useState in components, no global state library

## Environment Variables

Frontend (`.env`):

```
VITE_RECAPTCHA_SITE_KEY=...
VITE_API_BASE=http://localhost:3002
```

Backend (`backend/.env`):

```
PORT=3002
SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS, SMTP_EMAIL
VERIF_HMAC_KEY
ORCID_CLIENT_ID, ORCID_CLIENT_SECRET
RECAPTCHA_SECRET
SDX_API_BASE_URL=http://localhost:6098
```

## Key Components

- **Dashboard** (`src/components/pages/Dashboard.tsx`) - Main authenticated view with topology map and L2VPN management
- **TopologyMap** (`src/components/TopologyMap.tsx`) - Leaflet-based network visualization
- **NewL2VPNModal** (`src/components/NewL2VPNModal.tsx`) - L2VPN creation form with endpoint selection
- **TokenExpiryNotification** - Handles token refresh warnings

## Docker

```bash
# Build and start all services
docker-compose up --build -d

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild specific service
docker-compose build frontend
docker-compose build backend
```

**Container Architecture**:

- `frontend`: Nginx serving static React build (port 80)
- `backend`: Node.js Express server (port 3004)
- Nginx proxies `/api/*` and `/oauth/*` requests to backend

**Requirements**: Copy `backend/.env.example` to `backend/.env` and configure before running.

## Production Deployment

- Base path changes to `/multi-provider-authe` in production (see `vite.config.ts`)
- GitHub Pages deployment configured
- Docker deployment: Frontend on port 80, backend on port 3004

application runs on http://127.0.0.1:5002/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atlanticwave-sdx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/atlanticwave-sdx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
