## vercel001

> Next.js 16 + React 19 + TypeScript PWA app ("زاد مسلم"). Prayer times, Quran, Hadith, Azkar, push notifications.

# AGENTS.md — Zad Muslim

## Project
Next.js 16 + React 19 + TypeScript PWA app ("زاد مسلم"). Prayer times, Quran, Hadith, Azkar, push notifications.

**Deploy target**: Vercel. Client-side storage (IndexedDB/LocalStorage) — no server DB required in production.

## Commands
```
npm install          # installs deps + runs prisma generate (postinstall hook)
npm run dev          # next dev --webpack -p 3000 | tee dev.log
npm run build        # next build
npm run lint         # eslint .
npx playwright test  # e2e tests (requires running dev server on :3000)
```
Prisma (SQLite): `npm run db:push`, `npm run db:migrate`, `npm run db:reset`

## Architecture

### App shell
- **Entry**: `src/app/page.tsx` — single-page tab app (home, quran, prayer, azkar, more)
- **Layout**: `src/app/layout.tsx` — RTL Arabic root, dark theme, `<Providers>` wrapper
- **Providers**: `src/components/providers.tsx` — TanStack Query only (staleTime 30min)
- **Tabs**: state driven via `useAppStore.activeTab`; "more" tab uses `moreView` state for sub-views

### State
- **Zustand stores** in `src/store/`: `app-store`, `settings-store`, `quran-store`, `quran-cache-store`, `salawat-store`
- Settings persist in LocalStorage; goals/cache in IndexedDB

### Data layer
- **Prisma + SQLite** for push subscriptions, dhikr entries, and daily goals (`prisma/schema.prisma`)
- `src/lib/db.ts` — PrismaClient with dev query logging
- Run `npm run db:generate` after any schema change
- **API routes** in `src/app/api/`: `prayer`, `quran`, `azkar`, `hadith`, `radio`, `goals`, `push/*`, `sw-settings`

### i18n
- next-intl dependency present but **no `messages/` or `src/i18n/` directory found** — likely unused or inlined. App has manual AR/EN strings via `useSettingsStore().language`.

### Service Worker / PWA
- `@serwist/next` generates SW from `src/app/sw.ts` → `public/sw.js`
- Handles: precaching, runtime caching, prayer-time offline notifications, salawat reminders, push events
- Periodic sync tags: `prayer-check` (1min), `salawat-check` (5min)

### Styling
- **Tailwind CSS v4** — colors and theme defined in `src/app/globals.css` via `@theme` block
- `tailwind.config.ts` is kept for shadcn/ui compatibility only; colors come from CSS
- **shadcn/ui** components in `src/components/ui/`
- Custom color tokens: `zad-*` (zad-midnight, zad-gold, zad-surface, etc.)

## Important quirks
- **Many backup files** scattered throughout (`*.backup*`, `*.orig`, `*.rej`) — do NOT edit these
- `next.config.ts` has `typescript.ignoreBuildErrors: true` — typecheck separately with `npx tsc --noEmit`
- Dev server logs to `dev.log` via pipe
- `allowedDevOrigins` includes `http://192.168.1.3:3000` for LAN dev
- VAPID env vars needed for push notifications: `NEXT_PUBLIC_VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT`

## E2E tests
- Single spec: `e2e/comprehensive.spec.ts`
- Playwright config: `playwright.config.ts` — Chromium only, retries in CI, screenshots + video on
- Base URL: `http://localhost:3000` — dev server must be running

## Verification order
`lint → typecheck → build`
```
npm run lint && npx tsc --noEmit && npm run build
```

---
> Source: [deutch042/vercel001](https://github.com/deutch042/vercel001) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
