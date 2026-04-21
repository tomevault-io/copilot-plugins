## home-automation

> This is a home automation platform that provides a beautiful, modern web interface for managing and monitoring smart home devices. The goal is to create a unified dashboard that brings together multiple home automation systems.

# Home Automation - AI Worker Instructions

## Project Overview

This is a home automation platform that provides a beautiful, modern web interface for managing and monitoring smart home devices. The goal is to create a unified dashboard that brings together multiple home automation systems.

## Tech Stack

- **Runtime**: Node.js 20+ with TypeScript (strict mode)
- **Frontend**: React 18+ with Vite, Tailwind CSS
- **Backend**: Express.js REST API
- **Package Manager**: npm

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server (frontend + backend)
npm run build        # Production build
npm run typecheck    # TypeScript type checking
npm test             # Run tests (vitest, single pass)
npm run test:watch   # Run tests in watch mode
```

## Architecture

```
src/
  server/           # Express.js backend API
    integrations/   # Smart home platform connectors
      fibaro/       # Fibaro HC3 integration
    routes/         # API route handlers
    index.ts        # Server entry point
  client/           # React frontend
    components/     # UI components
    pages/          # Page-level views
    hooks/          # Custom React hooks
    services/       # API client services
    App.tsx         # Root component
    main.tsx        # Entry point
```

## Integrations

### Fibaro HC3

- **URL**: `http://192.168.1.35` (routed via Tailscale subnet routing through rsiw1)
- **Auth**: Basic auth using environment variables `FIBARO_USERNAME` and `FIBARO_PASSWORD`
- **API Docs**: Fibaro HC3 uses a REST API. Key endpoints:
  - `GET /api/devices` - List all devices
  - `GET /api/devices/{id}` - Device details
  - `POST /api/devices/{id}/action/{actionName}` - Trigger device action (e.g., `turnOn`, `turnOff`, `setValue`)
  - `GET /api/rooms` - List rooms
  - `GET /api/scenes` - List scenes
  - `POST /api/scenes/{id}/execute` - Execute a scene
  - `GET /api/weather` - Weather data
  - `GET /api/panels/location` - Location/sunrise/sunset info

### Netatmo

- **Status**: To be configured
- **Auth**: OAuth2 via environment variables `NETATMO_CLIENT_ID`, `NETATMO_CLIENT_SECRET`, `NETATMO_REFRESH_TOKEN`
- **API Docs**: https://dev.netatmo.com/apidocumentation

## Environment Variables

Create a `.env` file (never commit this):

```
FIBARO_URL=http://192.168.1.35
FIBARO_USERNAME=admin
FIBARO_PASSWORD=<set-in-env>
NETATMO_CLIENT_ID=<set-in-env>
NETATMO_CLIENT_SECRET=<set-in-env>
NETATMO_REFRESH_TOKEN=<set-in-env>
PORT=3000
API_TOKEN=<generate-with-openssl-rand-hex-32>
VITE_API_TOKEN=<same-value-as-API_TOKEN>
ALLOWED_ORIGIN=
```

`API_TOKEN` is required — the server will refuse to start without it. `VITE_API_TOKEN` must match `API_TOKEN`; it is embedded into the frontend bundle at build time and sent as a `Bearer` token on all `/api/fibaro` requests.

`FIBARO_URL`, `FIBARO_USERNAME`, and `FIBARO_PASSWORD` are also required — the Fibaro client validates all three on startup and exits with a fatal error if any are missing.

`ALLOWED_ORIGIN` is optional. When unset (or empty), CORS is configured to same-origin only (cross-origin requests are blocked by the browser). Set it to a specific origin (e.g., `http://localhost:5173`) to allow cross-origin requests from that origin during development.

## Design Guidelines

- **Modern and clean**: Use a dark theme with accent colors for status indicators
- **Responsive**: Must work well on desktop, tablet, and mobile
- **Real-time**: Show live device states, auto-refresh data
- **Intuitive**: Group devices by room, use clear icons, show at-a-glance status
- **Performance**: Minimize API calls with smart caching and WebSocket updates where possible

## Conventions

- Use functional React components with hooks
- Use TypeScript strict mode - no `any` types
- API responses should be typed with shared interfaces
- Error handling: wrap all API calls with proper error boundaries
- Use environment variables for all secrets - never hardcode credentials
- Write tests for integration connectors and critical UI logic
- Tests use **vitest**; client hooks (`src/client/hooks/**`) run in the `jsdom` environment (configured via `environmentMatchGlobs` in `vitest.config.ts`)

## Allowed Domains

- `127.0.0.1` (Fibaro HC3 proxy)
- `api.netatmo.com`
- `registry.npmjs.org`
- `github.com`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digifyno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
