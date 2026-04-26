## legacy-code-migration-to-ui-application

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Both frontend and backend must be started separately. Run each in its own terminal.

**Backend** (Express API on port 3001):
```bash
cd backend
npm install
npm run dev        # ts-node src/server.ts
npm run build      # tsc → dist/
npm start          # node dist/server.js
```

**Frontend** (Vite/React on port 5173):
```bash
cd frontend
npm install
npm run dev        # vite dev server
npm run build      # tsc && vite build
```

No linting or test scripts are configured. TypeScript type-checking runs as part of `build`.

## Architecture

This is a COBOL-to-web migration — a two-tier app with a React frontend and Express backend, each in its own npm workspace with its own `tsconfig.json`.

**Auth + data flow:**
```
App.tsx (auth router)
  ├── LoginPage.tsx   → users.ts (authenticate)
  └── Dashboard.tsx   → HTTP fetch → server.ts → operations.ts → dataProgram.ts
```

### Backend (`backend/src/`)
| File | COBOL equivalent | Purpose |
|------|-----------------|---------|
| `dataProgram.ts` | `data.cob` | In-memory balance store (initial: $1,000.00); resets on restart |
| `operations.ts` | `operations.cob` | `getBalance`, `credit`, `debit` business logic |
| `server.ts` | main menu dispatch | Express app; defines REST endpoints |

**API endpoints:**
- `GET /balance` → current balance
- `POST /credit` `{ amount: number }` → add to balance
- `POST /debit` `{ amount: number }` → subtract (fails with HTTP 422 if insufficient funds; HTTP 400 for non-positive amounts)

CORS is enabled on the backend to allow requests from the Vite dev server.

### Frontend (`frontend/src/`)
| File | Purpose |
|------|---------|
| `App.tsx` | Auth router — holds `User \| null` state; renders `LoginPage` or `Dashboard` |
| `users.ts` | Hardcoded `USERS` array + `authenticate(username, password)` helper |
| `LoginPage.tsx` | Two-panel split login: dark branding left, white form card right |
| `Dashboard.tsx` | Full-page app shell: nav, home tiles, per-view panels, profile dropdown |

**Dashboard navigation model** (`type View = 'home' | 'balance' | 'credit' | 'debit'`):
- Home → three action tiles (View Balance, Credit, Debit)
- Clicking a tile sets `view` and navigates to the panel; `← Back` returns to `'home'`
- Balance is fetched on demand (when the Balance tile is clicked), not on mount
- Profile dropdown (avatar chip in nav) shows display name, username, account number, and the last-known balance

**Auth model:** No JWT or session storage — `User` state lives in React memory. Logout clears state and returns to `LoginPage`.

TypeScript is set to `strict: true` in both workspaces. The backend compiles to CommonJS (`dist/`); the frontend uses ESNext modules via Vite's bundler resolution.

## Design system

All styles are inline `React.CSSProperties`. No CSS files or Tailwind.

| Token | Value | Usage |
|-------|-------|-------|
| Navy | `#0f172a` | Login left panel, balance card gradient, primary text |
| Blue | `#2563eb` | Primary action buttons, logo dot, avatar accent |
| Slate-50 | `#f8fafc` | Dashboard background, input fills |
| Slate-200 | `#e2e8f0` | Borders, dividers |
| Slate-500 | `#64748b` | Secondary text, labels |
| White | `#ffffff` | Cards, nav bar |

No red buttons on any action. Debit uses white + `#cbd5e1` border (secondary/cautious). Credit uses solid blue (primary/positive).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KunjMaheshwari) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
