## slotmachine

> 5-reel video slot machine web app. Learning/portfolio project. See `mvp.md` for full design spec.

# CLAUDE.md — SlotMachine Project

## Project Overview

5-reel video slot machine web app. Learning/portfolio project. See `mvp.md` for full design spec.

## Stack

- **Frontend**: React + Vite + TypeScript (`/frontend`)
- **Backend**: .NET 8 Minimal API (`/backend`)
- **Animation**: GSAP

## Running the App

```bash
# Backend (port 5000)
cd backend && dotnet run

# Frontend (port 5173)
cd frontend && npm run dev
```

## Project Structure

```
SlotMachine/
├── backend/
│   ├── Program.cs              # entry point, CORS, endpoint registration
│   ├── Models/
│   │   ├── SpinRequest.cs
│   │   └── SpinResult.cs
│   └── Services/
│       └── SlotService.cs      # RNG, symbol weights, win logic
├── frontend/
│   ├── src/
│   │   ├── api/slotApi.ts      # POST /api/spin fetch wrapper
│   │   ├── hooks/useSlotGame.ts # game state + spin handler
│   │   ├── components/
│   │   │   ├── Reel.tsx
│   │   │   ├── SlotMachine.tsx
│   │   │   ├── BetControls.tsx
│   │   │   ├── SpinButton.tsx
│   │   │   ├── BalanceDisplay.tsx
│   │   │   └── WinModal.tsx
│   │   └── App.tsx
│   └── vite.config.ts          # proxies /api → http://localhost:5000
├── mvp.md
└── CLAUDE.md
```

## Architecture Decisions

- **Game logic on backend**: RNG and win calculation happen in `SlotService.cs`. The frontend only animates the result it receives.
- **Stateless API**: No session, no database. Balance lives in React state.
- **Vite proxy**: `/api` calls are proxied to the backend in dev — no CORS issues, no hardcoded ports in the frontend.
- **GSAP for animation**: Reel spins use GSAP timelines for staggered stops (left to right, 0.3s gap).

## Key Conventions

- All game logic changes go in `SlotService.cs` — not in the controller or frontend
- Symbol payouts defined as a dictionary in `SlotService.cs`
- Paylines defined as arrays of `[reel, row]` coordinates
- React components are function components with TypeScript interfaces for all props
- API types mirrored in `src/api/slotApi.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hugo8791) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
