## 123123

> This file provides guidance to Claude Code (claude.ai/code) and other AI assistants when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and other AI assistants when working with code in this repository.

## Project Overview

A **mobile-first property repair-request system (房屋報修系統)** for tenants,
built with **React 18 + Vite**. Tenants open a web page, pick their name from a
list, attach photos, choose a category/urgency, describe the problem, and submit.
The landlord receives an Email and manages each request's status in an admin
dashboard. UI is in Traditional Chinese.

There is **no traditional backend**. The backend is a **Google Apps Script** Web
App bound to a Google Sheet (`google-apps-script/Code.gs`): it appends rows to the
sheet, saves photos to Google Drive, emails the landlord, and serves list/status
APIs. When `ENDPOINT_URL` is not configured, the app falls back to a **Demo mode**
that persists to `localStorage` so it runs without any backend.

## Tech Stack

- **React 18** (function components + hooks, no router library — hash-based routing)
- **Vite 6** (dev server + production bundler)
- Plain CSS, single mobile-first stylesheet (`src/index.css`, max-width 480px)
- Backend: **Google Apps Script** + **Google Sheets** + **Google Drive** (no server)

## Repository Structure

```
.
├── index.html                  # Vite entry HTML
├── vite.config.js              # Vite config (React plugin)
├── package.json                # scripts + dependencies
├── README.md                   # user-facing setup guide (中文)
├── src/
│   ├── main.jsx                # React entry
│   ├── config.js               # ⭐ all user-editable settings live here
│   ├── api.js                  # data layer: Apps Script fetch OR localStorage demo
│   ├── App.jsx                 # hash routing (#/ form, #/admin dashboard) + demo banner
│   └── pages/
│       ├── RepairForm.jsx      # tenant-facing repair form
│       └── AdminDashboard.jsx  # landlord dashboard (passcode-gated, status mgmt)
└── google-apps-script/
    ├── Code.gs                 # backend: doPost/doGet, sheet, Drive, email
    └── README.md               # backend deployment guide (中文)
```

## Development Workflow

- **Install:** `npm install`
- **Dev server:** `npm run dev` (http://localhost:5173, hot reload)
- **Build:** `npm run build` (outputs to `dist/`, static files)
- **Preview build:** `npm run preview`
- **Lint:** `npm run lint`

No test suite yet. If adding tests, prefer Vitest and document the command here.

## Key Architecture Notes

- **`src/config.js` is the single source of truth** for user configuration:
  `ENDPOINT_URL`, `TENANTS` (name → unit/phone), `CATEGORIES`, `URGENCIES`,
  `STATUSES`, `ADMIN_PASSCODE`, `MAX_PHOTOS`. Prefer adding new tunables here
  rather than hard-coding them in components.
- **`src/api.js` is the only place that talks to storage.** It branches on
  `isDemoMode` (true when `ENDPOINT_URL` is empty). `submitRequest`,
  `fetchRequests`, and `updateStatus` each have a localStorage path and an Apps
  Script `fetch` path. Keep both paths in sync when changing the data shape.
- **Request data shape:** `{ id, createdAt, status, tenantName, unit, phone,
  category, urgency, urgencyLabel, description, preferredTime, photos }`. Photos
  are base64 data URLs on submit; the Apps Script converts them to Drive links,
  so the admin view handles both string URLs and `{dataUrl}` objects.
- **CORS:** POSTs to Apps Script use `Content-Type: text/plain` deliberately to
  avoid a CORS preflight. Do not change this to `application/json`.
- **Routing** is hash-based in `App.jsx` (`#/admin` → dashboard). No router dep.
- **Apps Script changes** only take effect after redeploying a *new version* — note
  this whenever you edit `Code.gs`.

## Conventions

- Function components + hooks only; keep components focused on rendering and put
  data logic in `api.js` or `config.js`.
- Update state immutably (`map`/`filter`/spread).
- User-facing text is Traditional Chinese.
- Style with the CSS custom properties in `:root` (`--accent`, `--danger`, etc.)
  rather than hard-coding colors. Layout is constrained to a phone width.

## Git & Branching

- Active AI development branch: `claude/claude-md-docs-uwchm6`.
- Push with `git push -u origin <branch-name>`.
- Do not push to the default branch or open a PR without explicit permission.

## Notes for AI Assistants

- Keep this file in sync with the code. Run `npm run build` to verify before
  documenting commands.
- When changing the request data shape, update **all three** layers: the form
  (`RepairForm.jsx`), the data layer (`api.js` both modes), and the backend
  (`Code.gs` row columns + `listRequests`).

---
> Source: [YCMNick/123123](https://github.com/YCMNick/123123) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
