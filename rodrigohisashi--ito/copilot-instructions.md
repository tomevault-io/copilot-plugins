## ito

> ITO é um jogo de cartas cooperativo onde os jogadores recebem cartas numeradas (0-100) e devem organizá-las em ordem sem revelar seus números - apenas dando dicas criativas baseadas em um tema.

# ITO Game - Project Context

## Overview

ITO é um jogo de cartas cooperativo onde os jogadores recebem cartas numeradas (0-100) e devem organizá-las em ordem sem revelar seus números - apenas dando dicas criativas baseadas em um tema.

Este projeto é uma versão digital mobile-first para jogar com amigos.

## Tech Stack

```
Backend:   Node.js + Express + Socket.io
Frontend:  React + Vite + Tailwind CSS + Framer Motion
Database:  In-memory (no persistence needed)
Hosting:   Digital Ocean ($4 Droplet)
Language:  Portuguese (Brazil)
```

## Project Structure

```
ito-game/
├── server/
│   ├── index.js           # Express + Socket.io server
│   ├── gameLogic.js       # Game rules and card distribution
│   └── roomManager.js     # Room creation/management
├── client/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Card.jsx           # Animated card component
│   │   │   ├── PlayerList.jsx     # Players in room
│   │   │   ├── RoomCode.jsx       # Display/share room code
│   │   │   └── Button.jsx         # Reusable button
│   │   ├── pages/
│   │   │   ├── Home.jsx           # Create/join room
│   │   │   ├── Lobby.jsx          # Waiting room
│   │   │   └── Game.jsx           # Main game screen
│   │   ├── hooks/
│   │   │   └── useSocket.js       # Socket.io hook
│   │   ├── context/
│   │   │   └── GameContext.jsx    # Game state
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css              # Tailwind imports
│   ├── public/
│   │   └── sounds/                # Optional sound effects
│   ├── index.html
│   ├── tailwind.config.js
│   ├── vite.config.js
│   └── package.json
├── package.json                   # Root package.json
├── CLAUDE.md
├── PLAN.md
└── README.md
```

## Game Flow

1. **Home** → Host creates room, gets 4-digit code
2. **Join** → Players enter code and name
3. **Lobby** → Everyone waits, host sees player list
4. **Start** → Host clicks "Começar", cards are dealt (0-100)
5. **Play** → Each player sees only their card
6. **Reveal** → Host controls when to reveal order

## Key Features

- Mobile-first responsive design
- Real-time WebSocket communication
- Beautiful card animations (flip, reveal)
- Dark theme with gradients
- PWA installable
- No login required
- Room codes for easy joining
- Portuguese (Brazil) interface

## Socket Events

### Client → Server
- `create-room` → Create new game room
- `join-room` → Join existing room with code
- `start-game` → Host starts the game
- `reveal-card` → Player reveals their card
- `reset-game` → Start new round

### Server → Client
- `room-created` → Returns room code
- `player-joined` → New player notification
- `game-started` → Cards dealt to all
- `card-revealed` → Someone revealed card
- `error` → Error messages

## Design Guidelines

- **Colors**: Dark theme with purple/blue gradients
- **Cards**: Glassmorphism effect, smooth animations
- **Buttons**: Large, touch-friendly (min 48px)
- **Typography**: Clean, readable on mobile
- **Spacing**: Generous padding for fat fingers

## Commands

```bash
# Development
cd client && npm run dev      # Frontend on :5173
cd server && npm run dev      # Backend on :3001

# Production
npm run build                 # Build client
npm start                     # Start server (serves client)
```

## Environment Variables

```
PORT=3001                     # Server port
NODE_ENV=production           # production | development
```

## Deployment (Digital Ocean)

1. Create $4 Droplet (Ubuntu)
2. Install Node.js 20+
3. Clone repo
4. `npm install && npm run build`
5. Use PM2 for process management
6. Configure nginx as reverse proxy (optional)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrigohisashi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
