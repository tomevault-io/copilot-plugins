## ipt

> This repo is a **static web app** (no build step) plus **Playwright smoke tests**.

# Dev environment setup (cloud agents)

This repo is a **static web app** (no build step) plus **Playwright smoke tests**.

The goal of this doc is to make a fresh, ephemeral environment (cloud agents / CI) reproducible:
- install dependencies once
- start a static server
- run smoke tests headless

## Requirements

- **Node.js 20+**
- **npm 9+**

For Playwright on Linux (CI runners) you may need system deps. Prefer:

```bash
npx playwright install chromium --with-deps
```

## Quick start (Linux/macOS)

```bash
cd Volley-main
npm ci
npm run validate:static

# Run smoke tests (starts http-server automatically via Playwright webServer)
npx playwright install chromium --with-deps
npm run test:smoke
```

## Quick start (Windows)

```powershell
cd .\Volley-main
npm ci
npm run validate:static

npx playwright install chromium
npm run test:smoke
```

## Dev Containers / Codespaces (recommended for cloud agents)

This repo includes a ready-to-use `.devcontainer/`:

- opens with Node.js 20
- runs `npm ci`
- installs Playwright Chromium (with system deps)

After the container is ready:

```bash
npm run dev:serve
```

or

```bash
npm run test:smoke
```

## Running a dev server

This project is static, so any HTTP server works.

### Option A: Node (recommended)

```bash
cd Volley-main
npx http-server . -p 8000 -c-1
```

Open `http://127.0.0.1:8000`.

### Option B: Python

```bash
cd Volley-main
python3 -m http.server 8000
```

## Environment variables

Playwright uses these env vars (defaults are shown):

- `SMOKE_HOST` (default: `127.0.0.1`)
- `SMOKE_PORT` (default: `9011`)
- `CI` (when set, Playwright will not reuse an existing webServer)

Example:

```bash
SMOKE_HOST=127.0.0.1 SMOKE_PORT=8000 npx playwright test
```

## Common pitfalls

- **Service Worker**: avoid opening the app via `file://` in cloud environments; use HTTP.
- **Blocked external scripts**: if you add `integrity=` to CDN scripts, keep the SRI hash updated or Chromium will block the resource.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/t9923503503)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/t9923503503)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
