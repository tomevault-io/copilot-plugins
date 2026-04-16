## skydrift

> SkyDrift is a balloon constellation tracking application with Next.js frontend and Python FastAPI backend.

# SkyDrift Project Rules

## Project Overview
SkyDrift is a balloon constellation tracking application with Next.js frontend and Python FastAPI backend.

## Tech Stack
- Frontend: Next.js 14, React, TypeScript, Mapbox GL JS, TailwindCSS
- Backend: Python 3.11+, FastAPI, httpx
- Caching: In-memory with TTL

## Code Style

### TypeScript/React
- Use functional components with hooks
- Prefer named exports
- Use TypeScript strict mode
- Use SWR for data fetching
- Keep components small and focused
- Use TailwindCSS for styling

### Python
- Use type hints everywhere
- Use async/await for I/O operations
- Use Pydantic for data validation
- Keep functions small and focused
- Use dependency injection for services

## File Organization
- Frontend components in `frontend/src/components/`
- Frontend hooks in `frontend/src/hooks/`
- Backend services in `backend/app/services/`
- Backend routes in `backend/app/routers/`

## API Design
- RESTful endpoints
- Cache responses where appropriate
- Return consistent JSON structures
- Handle errors gracefully

## Performance
- Cache external API responses
- Use lazy loading for non-critical data
- Debounce user inputs
- Minimize re-renders

## Styling
- Light color scheme (white/light gray)
- Modern fonts: DM Sans, Space Mono
- Vibrant balloon colors
- Subtle shadows and rounded corners

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dghosh-2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
