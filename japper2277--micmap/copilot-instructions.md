## micmap

> **MicFinder NYC (MicMap)** is a web application for discovering comedy open mics in New York City.

# Gemini Context & Instructions

## Project Overview
**MicFinder NYC (MicMap)** is a web application for discovering comedy open mics in New York City.
It currently operates with a **Vanilla JavaScript frontend** (deployed to Netlify) and a **Node.js/Express backend** (deployed to Railway).

There is also a React-based frontend in `client/` ("P.U.R.E. MVP"), but the primary active development appears to be the Vanilla JS version in `map_designs/newest_map`.

## Architecture

### Frontend (Primary)
*   **Location:** `map_designs/newest_map/`
*   **Tech Stack:** Vanilla JavaScript, Tailwind CSS (CDN), Leaflet.js (CDN).
*   **Build:** None (Static HTML/JS).
*   **Key Files:**
    *   `index.html`: Main entry point. Loads libraries and scripts.
    *   `js/config.js`: Configuration constants.
    *   `js/state.js`: Global state management.
    *   `js/app.js`: Main initialization logic.
    *   `js/render.js`: Handles UI rendering (cards, lists).
    *   `js/map.js`: Leaflet map integration.

### Backend
*   **Location:** `api/`
*   **Tech Stack:** Node.js, Express.js.
*   **Database:** MongoDB (via Mongoose), Redis (Caching).
*   **External APIs:**
    *   MTA GTFS (Subway data).
    *   Google Distance Matrix & HERE API (Transit/Walking times).
    *   Google Sheets (Legacy/Seed data source).
*   **Key Files:**
    *   `server.js`: Main entry point.
    *   `models/`: Mongoose schemas.
    *   `routes/`: API route definitions (if separated, currently mainly in server.js or implied).

### React Frontend (Alternative/Future)
*   **Location:** `client/`
*   **Tech Stack:** React, Vite.
*   **Status:** "P.U.R.E. MVP" - Simpler list-based view.

## Development Workflow

### Starting the Project
**Frontend (Active):**
```bash
# Serve locally (requires live-server or similar)
npx live-server map_designs/newest_map
# OR simply open the file in a browser
open map_designs/newest_map/index.html
```

**Backend:**
```bash
cd api
npm install
npm start       # Runs node server.js (Port 3001)
npm run dev     # Runs with nodemon
```

### Testing
**Backend:**
```bash
cd api
npm test        # Runs Jest tests
```

## Coding Conventions

### General
*   **Style:** Follow existing patterns. Use `const`/`let`, arrow functions, and clear variable names.
*   **Comments:** Explain *complex* logic. Do not over-comment obvious code.

### Frontend (Vanilla JS)
*   **Modularization:** Code is split into multiple files in `js/` but loaded sequentially in `index.html`.
*   **State:** Use the global `STATE` object in `js/state.js`.
*   **DOM:** Direct manipulation or helper functions in `js/utils.js`. Avoid introducing frameworks like React/Vue into the `map_designs/newest_map` folder.

### Backend
*   **Async/Await:** Prefer async/await over callbacks.
*   **Error Handling:** Use proper error handling middleware.

## Important Notes
*   **Deployment:**
    *   Frontend: Netlify (Auto-deploys `map_designs/newest_map` from main branch).
    *   Backend: Railway (`https://micmap-production.up.railway.app`).
*   **Data Flow:** Frontend fetches from Backend (`/api/v1/mics`). Backend serves data from MongoDB/Redis.
*   **Mobile First:** The UI is heavily optimized for mobile (drawers, touch gestures). Always consider mobile responsiveness.

## Tool Usage
*   When asked to "fix a bug" or "add a feature", **always** check `map_designs/newest_map` first unless specified otherwise.
*   Use `search_file_content` to find relevant code snippets.
*   Use `read_file` to understand the context before editing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/japper2277)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/japper2277)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
