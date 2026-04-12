## fish-and-dragon

> Fish vs Dragon project overview and architecture


# Fish vs Dragon - Multiplayer Battle Game

This is a real-time multiplayer clicker game where Fish warriors cooperatively attack a Dragon.

## Project Structure
- `backend/` - Go backend with WebSocket support
- `frontend/` - React TypeScript frontend (development setup)
- `*.html` - Static HTML files served by Go backend
- `docker-compose.yml` - Container orchestration

## Key Technologies
- **Backend**: Go 1.21 with Gorilla WebSocket, Gorilla Mux, PostgreSQL
- **Frontend**: TypeScript, Bootstrap 5, WebSocket client
- **Database**: PostgreSQL with real-time state synchronization
- **Deployment**: Docker Compose

## Game Mechanics
- Players login and attack a shared Dragon with HP
- Dragon has exponential scaling: HP = baseHP * (1.4^level)
- Every 5 defeats increases dragon level
- All online players receive full rewards (no division)
- Two upgrade types: Sword (damage per click) and Auto-attack (background damage)

## Development Commands
- `docker-compose up --build` - Start full stack
- `cd backend && go run .` - Run backend only
- `cd frontend && npm start` - Run React frontend only

Always maintain the real-time multiplayer nature and cooperative gameplay mechanics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leftonbo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
