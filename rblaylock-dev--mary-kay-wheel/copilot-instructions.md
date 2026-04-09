## mary-kay-wheel

> A live spinning wheel app for Mary Kay makeup drawings. The host (mom) can add names, spin the wheel, and share the link so her Facebook group can watch the spin in real-time.

# Mary Kay Spinning Wheel

## Project Overview
A live spinning wheel app for Mary Kay makeup drawings. The host (mom) can add names, spin the wheel, and share the link so her Facebook group can watch the spin in real-time.

## Tech Stack
- **Framework**: Next.js 16 (App Router) + TypeScript
- **Styling**: Tailwind CSS v4 (CSS-based config via `@theme`)
- **Real-time**: Socket.IO (custom Express server)
- **No database** — rooms stored in memory (Map)
- **nanoid** generates short room codes

## How It Works
- **Host mode**: Visit `/` → click "Create a New Wheel" → get a shareable link. Host can add/remove names, spin the wheel.
- **Viewer mode**: Visit `/?room=XXXX` → watches the wheel live. Sees names, spin animation, and winner announcements in real-time.
- Socket.IO keeps host and all viewers in sync.

## Key Files
- `server.ts` — Custom Express server running Next.js + Socket.IO
- `src/app/page.tsx` — Main page component (manages state, Socket.IO events, host/viewer logic)
- `src/app/layout.tsx` — Root layout
- `src/app/globals.css` — Tailwind v4 theme config (Mary Kay colors) + custom utilities
- `src/components/Wheel.tsx` — Canvas-based spinning wheel
- `src/components/NamesList.tsx` — Add/remove names (host) or view names (viewer)
- `src/components/ShareLink.tsx` — Copyable room link for sharing
- `src/components/WinnerOverlay.tsx` — Winner announcement modal
- `src/components/History.tsx` — Past winners list
- `src/lib/socket.ts` — Socket.IO client singleton
- `src/lib/wheelColors.ts` — Mary Kay color palette for wheel slices
- `src/types/index.ts` — Shared TypeScript types

## Running Locally
```bash
npm install
npm run dev
# Open http://localhost:3000
```

## Building for Production
```bash
npm run build
npm start
```

## Status
- [x] Next.js 16 + Tailwind v4 + TypeScript setup
- [x] Custom server with Socket.IO (rooms, names, spins, winner sync)
- [x] React components: Wheel, NamesList, ShareLink, WinnerOverlay, History
- [x] Mary Kay branded color palette (pinks, rose gold, berry, plum, bronze)
- [x] Host mode (add names, spin, remove winners, share link)
- [x] Viewer mode (watch live, see names, see winner)
- [ ] Deploy somewhere publicly accessible (Railway, Render, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RBlaylock-Dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RBlaylock-Dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
