## blackjacksimulator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Blackjack Simulator with three main components:
1. Python backend API (FastAPI) for game logic and simulations
2. React frontend with Tailwind CSS for manual play
3. Reinforcement learning framework for betting strategy optimization (to be implemented)

## Development Commands

### Backend (Python/FastAPI)
```bash
# Install dependencies
cd backend
pip install -r requirements.txt

# Run the API server
cd backend/src
python api.py

# The API will be available at http://localhost:8000
```

### Frontend (React/Vite)
```bash
# Install dependencies
cd frontend
npm install

# Run development server
npm run dev

# Build for production
npm run build
```

## Architecture

### Backend Structure
- `backend/src/card.py`: Card and deck representations with Hi-Lo counting
- `backend/src/deck.py`: Deck management with configurable shuffle thresholds
- `backend/src/hand.py`: Hand logic including soft/hard values, splits, doubles
- `backend/src/game.py`: Core game state machine and rules engine
- `backend/src/api.py`: FastAPI endpoints for game sessions

### Frontend Structure
- `frontend/src/App.jsx`: Main game UI and state management
- `frontend/src/components/Card.jsx`: Card display component
- `frontend/src/components/Hand.jsx`: Hand display with game state indicators

### Key Design Decisions
- Session-based API design allows multiple concurrent games
- Game state machine ensures valid actions only
- Deck tracks running/true count for card counting strategies
- Frontend uses Vite for fast development and hot reload

## Next Implementation Steps

1. **Computer Player Strategies** (backend/src/strategy.py):
   - Basic strategy implementation
   - Configurable betting strategies
   - Card counting integration

2. **Reinforcement Learning** (backend/src/rl/):
   - State representation design
   - DQN or PPO agent implementation
   - Training loop and evaluation metrics

3. **Simulation Engine** (backend/src/simulator.py):
   - Batch game execution
   - Statistical analysis
   - Strategy comparison tools

4. **Enhanced UI Features**:
   - Strategy selection interface
   - Real-time statistics dashboard
   - Training visualization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AJM10565) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
