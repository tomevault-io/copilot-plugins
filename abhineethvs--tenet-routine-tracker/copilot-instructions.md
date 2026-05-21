## tenet-routine-tracker

> Authoritative plan: `MASTERPLAN.md`. API/env details: `REFERENCE.md`.

# TENET — project rules for Cursor

Authoritative plan: `MASTERPLAN.md`. API/env details: `REFERENCE.md`.

## Stack
- React + Vite, JavaScript (JSX), Tailwind CSS, `vite-plugin-pwa`.
- Memory: `localStorage` only (`src/store/memory.js` patterns per masterplan).

## Anthropic / API
- **Never** call `https://api.anthropic.com` from browser code.
- All Claude traffic from `src/api/claude.js` goes to **`POST /api/chat`** (same origin).
- API key: **`ANTHROPIC_API_KEY`** server-side only (Vercel env + local `.env`). **Do not** use `VITE_*` for the key.
- Proxy implementation lives in **`api/chat.js`** (Vercel serverless): forward body to Anthropic Messages API, return response.

## Secrets & git
- `.env` must stay out of git (see `.gitignore`). Use `.env.example` as the template.

## Scope
- Solo PWA; avoid extra backend, auth, DB, analytics, push notifications unless the masterplan changes.
- Prefer small, focused changes; match existing file style and naming.

---
> Source: [AbhineethVS/Tenet-routine-tracker](https://github.com/AbhineethVS/Tenet-routine-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
