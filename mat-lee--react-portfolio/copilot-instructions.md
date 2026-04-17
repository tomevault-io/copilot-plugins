## react-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Monorepo containing:
- **Portfolio** (codebymatthewlee.com) - React/Vite personal website
- **Labs** (labs.codebymatthewlee.com) - Docusaurus site for interactive explorations
- **Demos** - Shared React components used by both sites
- **API** (api.codebymatthewlee.com) - FastAPI backend for computational demos

## Commands

### Full Stack Development
```bash
pnpm install                 # Install all workspace dependencies
pnpm dev                     # Start all apps in parallel
pnpm dev:portfolio           # Start only portfolio (localhost:5173)
pnpm dev:labs                # Start only labs (localhost:3000)
```

### Backend (FastAPI)
```bash
cd server
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app:app --reload     # Starts on localhost:8000
```

### Building
```bash
pnpm build                   # Build all apps
pnpm build:portfolio         # Build only portfolio
pnpm build:labs              # Build only labs
```

## Project Structure

```
my-portfolio/
├── apps/
│   ├── portfolio/           # React/Vite main site
│   │   └── src/
│   │       ├── App.jsx      # Main portfolio page
│   │       └── LabPage.jsx  # Legacy lab renderer (uses @portfolio/demos)
│   └── labs/                # Docusaurus site
│       └── docs/            # MDX lab pages
├── packages/
│   └── demos/               # Shared React components
│       └── src/
│           ├── index.js     # Barrel export
│           └── TetrisMoveVisualizer/
│               ├── TetrisMoveVisualizer.jsx
│               ├── api.js   # Configurable API URL
│               ├── PiecePicker.jsx
│               └── TetrominoIcon.jsx
└── server/                  # FastAPI backend
    ├── app.py               # API with CORS for all domains
    ├── movegen.py           # Move generation algorithms
    └── tetris_logic.py      # Tetris game logic
```

## Adding a New Lab

1. **Create component** in `packages/demos/src/NewDemo/`
2. **Export** from `packages/demos/src/index.js`:
   ```js
   export { default as NewDemo } from './NewDemo/index.jsx';
   ```
3. **Create MDX** in `apps/labs/docs/`:
   ```mdx
   import { NewDemo } from '@portfolio/demos';

   # My Lab

   <NewDemo />
   ```

## API Configuration

The demos package reads API URL from:
1. `VITE_API_URL` env var (Vite apps)
2. Localhost fallback for development
3. `https://api.codebymatthewlee.com` for production

CORS is configured in `server/app.py` for all frontend domains.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mat-lee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
