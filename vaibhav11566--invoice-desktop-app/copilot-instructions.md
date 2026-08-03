## invoice-desktop-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An Electron desktop app for invoice management. The app embeds a React frontend (Vite) and an Express/MongoDB backend — all three layers run as a single desktop application.

## Development Commands

**Run all three layers concurrently (recommended for development):**
```bash
npm run dev        # from root — starts backend, frontend dev server, and Electron
```

**Run individual layers:**
```bash
npm run backend    # starts backend with nodemon (cd backend && npm run dev)
npm run frontend   # starts Vite dev server at http://localhost:5173
```

**Production build:**
```bash
npm run build      # builds frontend to frontend/dist/
npm start          # builds frontend then launches Electron loading the built files
```

**Backend only:**
```bash
cd backend && npm run dev    # nodemon
cd backend && npm start      # node directly
```

**Install dependencies** (must be done in all three locations):
```bash
npm install                  # root (Electron + concurrently)
cd backend && npm install
cd frontend && npm install
```

## Architecture

The app has three layers with distinct responsibilities:

### Electron (`electron-main.js`, `electron-preload.js`)
- `electron-main.js` spawns the backend as a child process (`node server.js`) before creating the browser window. It kills that process on quit.
- In dev mode (`NODE_ENV !== "production"`), Electron loads `http://localhost:5173` (Vite dev server). In production it loads `frontend/dist/index.html`.
- Context isolation is enabled; `nodeIntegration` is disabled. The preload script is minimal.

### Backend (`backend/`)
- Express server on port **5001** (set in `backend/.env`).
- MongoDB via Mongoose; connection string defaults to `mongodb://localhost:27017/invoice_db`.
- Two route groups: `/api/auth` and `/api/invoices`.
- File uploads (payment screenshots) handled by `multer`; served statically at `/uploads`.
- Auth uses JWT (`jsonwebtoken`). The `protect` middleware validates the Bearer token. `adminOnly` middleware gates invoice deletion to `role: "admin"` users.
- Required env vars in `backend/.env`: `PORT`, `MONGODB_URI`, `JWT_SECRET`, `NODE_ENV`.

### Frontend (`frontend/`)
- React 18 + React Router v6 + Vite.
- Vite dev server proxies `/api` and `/uploads` to `http://localhost:5001`, so the frontend always calls relative `/api/...` paths.
- Auth state is managed in `AuthContext` (`frontend/src/context/AuthContext.jsx`). JWT and user object are persisted to `localStorage`. A 401 response from the API auto-redirects to `/login`.
- `frontend/src/api/axios.js` is the single Axios instance — always import this instead of raw `axios` to get the token interceptor.

## Data Models

**User**: `name`, `email` (unique), `password` (bcrypt-hashed), `phone`, `role` (`admin`|`user`), `is_active`. Password is stripped from JSON output via `toJSON()`.

**Invoice**: Auto-generates `invoice_number` (`INV-YYYYMMDD-XXXXXXXX`) on first save. Contains line items with auto-calculated `total`, `subtotal`, `tax_amount`, and `total_amount`. Key status fields: `order_status` (Pending → Processing → Shipped → Delivered → Cancelled), `payment_status` (Pending | Verified | Failed), `payment_service` (UPI | BankTransfer | Cash | Card | Other). Soft-delete via `deleted_at` field.

## Ports & Proxy

| Service | Port |
|---------|------|
| Backend (Express) | 5001 |
| Frontend (Vite dev) | 5173 |
| Electron window | loads 5173 in dev, `dist/index.html` in prod |

## Frontend Routing

All invoice routes are under the `<Layout />` shell and require authentication:

| Path | Component |
|------|-----------|
| `/invoices` | `InvoiceList` |
| `/invoices/create` | `CreateInvoice` |
| `/invoices/:id` | `InvoiceDetail` |
| `/invoices/:id/edit` | `EditInvoice` |
| `/login`, `/register` | Public only (redirects to `/invoices` if logged in) |

---
> Source: [Vaibhav11566/invoice-desktop-app](https://github.com/Vaibhav11566/invoice-desktop-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
