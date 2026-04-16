## aros-developer-portal

> Port: 5442 | Protocol: HTTP | Dir: aros-developer-portal/

# aros-developer-portal

Port: 5442 | Protocol: HTTP | Dir: aros-developer-portal/

## Purpose
Public-facing AROS Developer Portal — documentation site for POS integration SDKs. React + Vite SPA served by an Express production server.

## Key Files
- `portal/server.mjs` — Production server (Express). Serves built SPA from `dist/`, provides `/health` + `/readyz`.
- `portal/src/App.jsx` — Main React app with SDK documentation
- `portal/vite.config.js` — Vite config (dev + build)
- `sdks/` — SDK source for JS, Python, C#, Swift, Kotlin, REST (OpenAPI)
- `examples/` — Integration examples for various POS systems

## Running
```bash
cd portal
npm install
npm run build       # Build for production
npm run serve       # Start production server on :5442
npm run dev         # Vite dev server on :5442
```

## Deployment
- LaunchAgent: `~/Library/LaunchAgents/ai.shre.developer-portal.plist`
- Launcher: `~/.local/bin/aros-developer-portal.sh`
- Domain: `developers.nirtek.net` (via Cloudflare tunnel -> shre-auth-gate -> :5442)
- Logs: `/tmp/aros-developer-portal.log`

## Standards
- Binds to `0.0.0.0` (platform standard)
- Port sourced from `ports.json` (single source of truth)
- Browser-first SPA — cannot import `shre-sdk` at runtime
- Health endpoints: `GET /health`, `GET /readyz`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nirpat3) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
