## hiko-pos

> - Frontend: React 18 + Vite 6, Tailwind CSS, Redux Toolkit, React Query, React Router

# Project Rules for Cursor (Restaurant POS System)

## Overview
- Stack: MERN
  - Frontend: React 18 + Vite 6, Tailwind CSS, Redux Toolkit, React Query, React Router
  - Backend: Node.js + Express, Mongoose, JWT, Razorpay
- Monorepo structure:
  - `pos-frontend/` (Vite app)
  - `pos-backend/` (Express API, **TypeScript** — source `*.ts`, compile output `dist/`)

## Run & Dev Commands
- Frontend
  - Install: `cd pos-frontend && npm install`
  - Dev: `cd pos-frontend && npm run dev` (serves at `http://localhost:5173`)
  - Build: `cd pos-frontend && npm run build`
  - Preview: `cd pos-frontend && npm run preview`
- Backend
  - Install: `cd pos-backend && npm install`
  - Dev: `cd pos-backend && npm run dev` (`tsx watch app.ts`, Express on `http://localhost:3000` by default)
  - Build: `cd pos-backend && npm run build` (`tsc` → `dist/`)
  - Start (production): `cd pos-backend && npm run build && npm start` (`node dist/app.js`)

## Environment
- Backend env (see `pos-backend/config/config.ts`):
  - `PORT` (default 3000)
  - `MONGODB_URI`
  - `NODE_ENV`
  - `JWT_SECRET`
  - `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET`
- Frontend dev server: `http://localhost:5173`
- CORS: backend allows origin `http://localhost:5173`

## API Base URLs
- Local dev: `http://localhost:3000/api`
- Routes:
  - `/api/user`
  - `/api/order`
  - `/api/table`
  - `/api/payment`

## Coding Conventions
- General
  - Use meaningful, descriptive names; avoid abbreviations
  - Prefer early returns, minimal nesting, and explicit error handling
  - Keep functions small and cohesive
- Frontend
  - React functional components, hooks
  - State: use Redux Toolkit slices in `pos-frontend/src/redux/slices/`
  - Data fetching/caching: RTK Query 
  - Routing: `react-router-dom@7`
  - Styling: Tailwind CSS; avoid inline styles unless necessary
- Backend
  - Route -> Controller -> Model pattern
  - Centralized error handling via `middlewares/globalErrorHandler.ts`
  - JWT auth middleware in `middlewares/tokenVerification.ts`
  - Mongoose models in `models/`

## File/Folder Guidelines
- Frontend
  - Components live under `src/components/*`
  - Pages under `src/pages/*`
  - Shared UI in `src/components/shared/*`
  - Keep constants in `src/constants/index.js`
  - HTTP helpers in `src/https/*`
  - Utilities in `src/utils/index.js`
- Backend
  - Endpoints registered in `app.ts` (compiled to `dist/app.js`)
  - Add new routes under `routes/`, controllers under `controllers/`, models under `models/`

## Testing & Quality
- Lint frontend: `cd pos-frontend && npm run lint`
- Prefer unit tests for pure utilities; integration tests for API endpoints (if/when added)

## Assistant Behavior (Cursor)
- Respect existing indentation and formatting; do not mass-reformat unrelated code
- When editing, touch only what is necessary; keep edits minimal and focused
- Add missing imports explicitly; avoid unused imports
- Keep secrets out of code; use environment variables
- For API integration in frontend, centralize base URL and interceptors in `src/https/`
- After edits involving build or runtime, run the appropriate dev/build command

## Common Tasks
- Add a new API endpoint
  1. Create model (if needed) in `pos-backend/models/`
  2. Create controller in `pos-backend/controllers/`
  3. Register route in `pos-backend/routes/` and mount in `app.ts`
  4. Update frontend service calls in `src/https/*`
- Add a new page in frontend
  1. Create page under `src/pages/`
  2. Add route in `src/pages/index.js` or router configuration
  3. Compose with components from `src/components/*`

## Ports and Networking
- Frontend: 5173
- Backend: 3000 (configurable via `PORT`)
- Ensure CORS `origin` matches frontend dev URL

## Notes
- Use absolute imports only if configured in Vite; otherwise prefer relative module paths
- Keep README updated for any changes to setup or commands 

---
> Source: [Khoanguyen0109/Hiko-POS](https://github.com/Khoanguyen0109/Hiko-POS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
