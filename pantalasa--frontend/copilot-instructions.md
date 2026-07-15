## frontend

> The frontend is a Node.js single-page web application that serves the Pantalasa

# Frontend — Agent Instructions

## Project Overview

The frontend is a Node.js single-page web application that serves the Pantalasa
merchant dashboard. It talks to the backend REST API and renders the operator UI.

**Tech stack:** Node.js 19, JavaScript, Docker
**Primary framework:** lightweight Node HTTP server (`src/index.js`)

## Architecture

- `src/` — Application source (entrypoint `src/index.js`)
- `package.json` — Dependencies and npm scripts
- `Dockerfile` — Container build
- `.argocd/` — ArgoCD Application manifest for deployment

## Build Commands

```bash
# Install dependencies
npm install

# Run tests
npm test

# Start locally
node src/index.js

# Build Docker image
docker build -t frontend:latest .
```

## Testing

- Unit tests run via `npm test` (jest).
- Add tests alongside source under `src/`.

## Code Style

- Standard JavaScript conventions; keep modules small and focused.
- Prefer async/await over nested callbacks.
- Keep UI state handling isolated from network calls.

---
> Source: [pantalasa/frontend](https://github.com/pantalasa/frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
