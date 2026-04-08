## candyclash

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Candy Clash is a competitive HTML5 match-3 game prototype where players compete for Gold Bar prize pools. Built with React/Phaser (frontend) and Node.js/Fastify (backend), deployed on Firebase.

## Tech Stack

- **Frontend:** React + Vite + TypeScript + Tailwind CSS + Phaser 3
- **Backend:** Node.js + TypeScript + Fastify + Socket.IO
- **Database:** Firebase Firestore (primary) + Firebase Realtime Database (leaderboards)
- **Hosting:** Firebase Hosting (frontend) + Firebase Cloud Functions (backend)
- **Auth:** Email-based dev login with JWT tokens

## Essential Commands

### Development
```bash
# Install dependencies (uses pnpm workspaces)
pnpm install

# Run both frontend and backend in dev mode
pnpm dev

# Run frontend only
cd app/frontend && pnpm dev

# Run backend only  
cd app/backend && pnpm dev
```

### Build & Deploy
```bash
# Build both frontend and backend
pnpm build

# Deploy to Firebase (builds first)
pnpm deploy

# Deploy specific services
firebase deploy --only hosting        # Frontend only
firebase deploy --only functions      # Backend only
firebase deploy --only firestore      # Firestore rules
```

### Testing & Maintenance
```bash
# Run linting
pnpm lint

# Format code
pnpm format

# Run backend scripts (from app/backend)
node scripts/check-database.js        # Check database state
node scripts/fix-global-stats.js      # Recalculate player statistics
node scripts/pay-all-tournaments.js   # Process retroactive payouts
node scripts/remove-users.js          # Remove problematic users
```

## Architecture

### Frontend Structure
```
app/frontend/src/
├── game/           # Phaser game logic
│   ├── Match3Scene.ts    # Main game scene
│   ├── types.ts          # Game type definitions
│   └── utils.ts          # Game utilities
├── components/     # React UI components
├── pages/         # Route pages
├── stores/        # Zustand state management
└── api/           # API client functions
```

### Backend Structure  
```
app/backend/src/
├── index.ts       # Fastify server setup
├── auth.ts        # JWT authentication
├── challenge.ts   # Tournament management
├── attempts.ts    # Game attempt validation
├── leaderboard.ts # Leaderboard operations
├── firebase.ts    # Firebase services
├── scheduled.ts   # Cloud Functions (auto-close tournaments)
└── utils/
    └── payout-calculator.ts  # Prize distribution logic
```

## Key Data Flows

### Tournament Flow
1. User joins challenge → Entry fee deducted → Attempt created
2. Game played in Phaser → Score submitted with attemptToken
3. Server validates → Updates leaderboard → Returns rank
4. Tournament expires → Scheduled function calculates payouts → Balances updated

### Authentication
- Email login → JWT issued → Token stored in localStorage
- All API calls include `Authorization: Bearer <token>`
- User ID extracted from JWT for operations

## Firebase Services

### Firestore Collections
- `users` - Player accounts and balances
- `challenges` - Tournament configurations  
- `attempts` - Game attempts and scores
- `transactions` - Gold bar transactions
- `globalStats` - Persistent player statistics
- `levels` - Level configurations

### Realtime Database Paths
- `/leaderboard/{challengeId}` - Live tournament rankings
- `/globalLeaderboard/{userId}` - Global player rankings
- `/pots/{challengeId}` - Current prize pools

### Cloud Functions
- `api` - Main backend API (Fastify)
- `autoCloseChallenges` - Hourly tournament closure (scheduled)

## Prize Distribution System

The payout calculator supports flexible prize structures:
- **Percentage-based:** Winners get % of pot (e.g., 40/25/15%)
- **Fixed amounts:** Guaranteed prizes regardless of pot
- **Range-based:** Positions 4-50 share prize pool
- **Minimum player requirements:** Refunds if too few players

Configuration in `challenge.prize_distribution`:
```javascript
{
  type: 'percentage',
  rules: [
    { position: 1, type: 'percentage', amount: 40 },
    { position: 2, type: 'percentage', amount: 25 },
    { position: 3, type: 'percentage', amount: 15 },
    { range: [4, 50], type: 'percentage', amount: 20, split: true }
  ],
  minimum_players: 3,
  refund_on_insufficient: true
}
```

## Game Mechanics

### Match-3 Core
- 8×8 grid with 5-6 candy colors
- Special candies: Striped (4-match), Wrapped (L-shape), Color Bomb (5-match)
- Special combinations multiply effects
- Objectives: Collect specific candies, score targets, time challenges

### Anti-Cheat
- Server-authoritative timers via attemptToken HMAC
- Move sequence validation
- Theoretical score limits
- Rate limiting on submissions

## Common Operations

### Update Daily Challenge
1. Modify challenge in Firebase Console or via admin API
2. Set new level_id, entry_fee, attempts_per_day, prize_distribution
3. Challenge auto-closes at ends_at timestamp

### Process Manual Payouts
```bash
cd app/backend
node scripts/pay-all-tournaments.js
```

### Fix Player Statistics
```bash
cd app/backend
node scripts/fix-global-stats.js
```

### Check Tournament Status
```bash
cd app/backend
node scripts/check-database.js
```

## Environment Variables

Required in `app/backend/.env`:
```
JWT_SECRET=your-secret-key
FRONTEND_ORIGIN=http://localhost:5173
NODE_ENV=development
```

Firebase credentials are handled via Application Default Credentials or service account JSON.

## Development Guidelines

### State Management
- Frontend: Zustand stores for auth, game state
- Backend: Firestore for persistence, Realtime DB for live data
- Never store sensitive data in localStorage

### Error Handling
- API returns `{ error: string }` format
- Frontend shows toast notifications for errors
- Backend logs errors with context

### Performance
- Batch Firestore operations when possible
- Use Realtime Database for frequently-read data
- Cache leaderboard queries
- Limit animation complexity on mobile

### Security
- Validate all inputs with zod/TypeScript
- Use Firebase Security Rules for database access
- Rate limit API endpoints
- Never expose admin endpoints without API key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenPomme)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BenPomme)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
