## frontend

> chop is a SoundCloud mix scanner and track identifier. This repo is the **web frontend** — the main product. Users sign up, manage their account, browse their full scan history and collections, play back sets and identified tracks, and handle billing here.

# chop — Frontend

## What this is
chop is a SoundCloud mix scanner and track identifier. This repo is the **web frontend** — the main product. Users sign up, manage their account, browse their full scan history and collections, play back sets and identified tracks, and handle billing here.

The browser extension triggers scans and stores results. This app is where users come to explore and manage everything.

- Backend repo: `chop-backend`
- Extension repo: `chop-extension`

---

## Stack
- **Next.js** (App Router, TypeScript)
- **Bun** — package manager
- **CSS Modules** — styling
- **Auth** — Supabase (@supabase/ssr)
- **Data fetching** — TanStack React Query
- **UI primitives** — Radix UI
- **Payments** — TBD (e.g. Stripe)

---

## Project structure
```
app/
├── (auth)/
│   └── login/
├── (home)/
│   └── components/
│       ├── hero.tsx               # hero section with scanner
│       └── recently-scanned.tsx   # displays recent scans
├── layout.tsx                     # root layout (Header, QueryProvider)
components/
├── header/                        # navigation header
├── logo/
├── scanner/
│   └── scanner.tsx                # client component — submit SoundCloud URLs
├── ui/                            # Radix-based primitives (button, input, nav)
└── providers/
    └── query-provider.tsx
hooks/
└── useScanner.ts                  # scan state + polling logic
lib/
├── api/
│   └── client.ts                  # all backend API calls
├── supabase/                      # Supabase auth (client, server, proxy)
└── utils.ts
types/
└── index.ts
```

---

## Players
Two distinct players:

- **Set player** — plays back full SoundCloud mixes. Uses the SoundCloud widget to stream the mp3.
- **Track player** — plays back individual identified tracks. Sources audio from Deezer.

---

## Backend API
Base URL via env:
```
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
```

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/scan` | Submit a SoundCloud URL for scanning |
| GET | `/sets` | List all past scans |
| GET | `/sets/{id}` | Single scan with identified tracks |

Always use `lib/api/client.ts` for all backend calls — never fetch directly from components.

---

## Types
Keep in sync with backend response shapes:

```ts
type Track = {
  id: string
  title: string
  artist: string
  timestamp: number // seconds into the mix
}

type SetMix = {
  id: number
  url: string           // SoundCloud URL
  title: string
  artist: string
  image_url: string | null
  created_at: string
}
```

---

## Conventions
- CSS Modules only — no Tailwind, no inline styles
- Server Components by default — only use `"use client"` when necessary
- All backend API calls go through `lib/api/client.ts`
- TypeScript strict mode — no `any`

---

## Running locally
```bash
bun dev
```

Make sure the backend is running at `localhost:8000` first.

---

## Build phases
- [x] Phase 1 — Scaffold + App Router layout + auth pages (login/register)
- [x] Phase 2 — Scanner component + recently scanned display
- [ ] Phase 3 — Auth completion + route protection
- [ ] Phase 4 — Set player (SoundCloud widget/mp3) + Track player (Deezer)
- [ ] Phase 5 — Settings + billing (Stripe)
- [ ] Phase 6 — Polish, empty states, loading states, PWA config

---
> Source: [heckspoiler/frontend](https://github.com/heckspoiler/frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
