## hackathon-prizepicks-submission

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **NBA Player Stock Market Trading Simulation** built as a monorepo with:
- **Backend**: Node.js/Express/Socket.IO real-time server (port 3001)
- **Frontend**: React Native/Expo cross-platform mobile app
- **Shared**: TypeScript type definitions shared between frontend/backend
- **Database**: Supabase with demo/mock mode fallback

## Development Commands

### Quick Start
```bash
# Install all dependencies
npm run install:all

# Start both backend and frontend concurrently
npm run dev

# Individual services
npm run backend:dev      # Backend with nodemon on port 3001
npm run frontend:start   # Expo dev server

# Additional workspace commands from root
npm run clean             # Remove all node_modules
npm run build:backend     # Build backend only
npm run start:backend     # Start production backend
npm run test:backend      # Backend tests only
npm run test:frontend     # Frontend tests only
```

### Individual Package Commands
```bash
# Backend
cd backend
npm run dev              # Development with nodemon
npm run build            # TypeScript compilation
npm run start            # Production build
npm test                 # Jest tests

# Frontend
cd frontend
npm start                # Expo dev server
npm run android          # Open Android emulator
npm run ios              # Open iOS simulator
npm run web              # Open web browser
npm test                 # Jest tests

# Shared Types
cd shared
npm run build            # Compile TypeScript types
npm run dev              # Watch mode compilation
```

### Environment Setup
Backend requires `.env` file (see `backend/.env.example`):
- PORT=3001
- CORS_ORIGIN=http://localhost:19006
- JWT_SECRET=your-secret-key
- SUPABASE_URL and SUPABASE_ANON_KEY (optional for demo mode)

Frontend configuration in `app.json` sets API URLs to localhost:3002, but backend runs on 3001 by default. Verify port consistency between backend/.env and frontend app.json.

## Architecture

### Monorepo Structure
```
├── backend/             # Node.js Express + Socket.IO server
│   ├── src/routes/     # REST API endpoints (/api/*)
│   ├── src/socket/     # Real-time handlers & game simulation
│   ├── src/data/       # Mock data & price engine integration
│   ├── src/services/   # Auth, database services
│   └── src/utils/      # PriceEngine class
├── frontend/           # React Native Expo app
│   ├── src/screens/    # Main app screens (4 tabs)
│   ├── src/context/    # React Context providers
│   ├── src/services/   # API client
│   └── src/theme/      # React Native Paper theming
└── shared/             # Shared TypeScript types
    └── src/types.ts    # Player, Portfolio, Trade interfaces
```

### Key Components

**PriceEngine** (`backend/src/utils/priceEngine.ts`):
- Singleton class managing real-time price updates
- Player-specific volatility simulation
- Integrates with Socket.IO for live price broadcasts

**Socket.IO Integration** (`backend/src/socket/`):
- Real-time trading, chat, price updates
- Game simulation with pre-scripted events
- Flash multipliers and market events

**React Context Architecture** (`frontend/src/context/`):
- `SocketContext`: WebSocket connection management
- `PortfolioContext`: User portfolio state
- `GameContext`: Live game state and trading

**API Structure** (`backend/src/routes/`):
- `/api/players` - Player data and price history
- `/api/portfolio` - User holdings and performance
- `/api/trades` - Trading execution and history
- `/api/leaderboard` - Rankings and statistics
- `/api/game` - Live game status and simulation

### Real-time Features
- **Live Trading**: Limited trades during simulated games
- **Price Updates**: 8-12 second intervals with volatility
- **Flash Multipliers**: Dynamic price boosts (15% chance per cycle)
- **Game Events**: Pre-scripted NBA events affecting prices
- **Chat System**: Real-time messaging during live games

### Data Models
Core types in `shared/src/types.ts`:
- `Player`: NBA player with stats, price history, volatility
- `Portfolio`: Season/live holdings with P&L tracking
- `Trade`: Market orders with execution details
- `LiveGame`: Game simulation state and events

### Development Notes
- TypeScript strict mode enabled in frontend, disabled in backend
- React Native Paper for Material Design components
- Path aliases configured (`@/*`, `@components/*`, etc.)
- Socket rooms for user-specific and game-specific events
- Supabase integration with demo mode fallback
- Mock data system with 10 NBA players and realistic price history
- **Shared types**: When modifying types in `shared/src/types.ts`, run `cd shared && npm run build` to recompile before testing
- Import shared types: `import { Player, Portfolio, Trade } from '@player-stock-market/shared'`

### Testing Strategy
Both frontend and backend have Jest configured:
- Backend: API endpoint testing
- Frontend: React Native component testing
- Use `npm test` in respective directories

### Mobile Development
- Expo managed workflow for easy cross-platform development
- React Navigation with bottom tabs (Portfolio, Live Trading, Leaderboard, Profile)
- React Native Paper theme customization in `frontend/src/theme/theme.ts`
- Platform-specific styling using React Native Platform API

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlouisugbo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
