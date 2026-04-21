## teamup

> TeamUp (branded as "WebPadel" / "We Team Up") is a web application for padel and sports players to find partners, organize events, and build community. Users create/join events (max 4 players typical for padel), manage friends, share post-event memories, and browse venue locations. The app is focused on the Lithuanian market (Vilnius-area venues).

# CLAUDE.md - TeamUp (WebPadel) Codebase Guide

## Project Overview

TeamUp (branded as "WebPadel" / "We Team Up") is a web application for padel and sports players to find partners, organize events, and build community. Users create/join events (max 4 players typical for padel), manage friends, share post-event memories, and browse venue locations. The app is focused on the Lithuanian market (Vilnius-area venues).

**Live URL:** https://teamup.lt / https://weteamup.app

## Tech Stack

- **Frontend:** React 18.2 + TypeScript (strict mode), bootstrapped with Create React App via CRACO
- **Styling:** Tailwind CSS 3.4 + Material-UI 5 + custom CSS variables (shadcn/ui-style HSL theming)
- **Routing:** React Router DOM 6
- **Backend/Database:** Supabase (PostgreSQL, Auth, Storage, Realtime) — migrating from Firebase
- **Email:** SendGrid (via Vercel API route at `api/send-email.ts`)
- **Maps:** MapLibre GL / React Map GL with MapTiler geocoding
- **i18n:** i18next (minimal setup, English only currently)
- **Build tool:** CRACO (wraps react-scripts)
- **Deployment:** Vercel (static build + serverless API routes)

## Commands

```bash
npm start          # Start dev server (CRACO)
npm run build      # Production build (CI=false used in CI to ignore warnings)
npm test           # Run tests (Jest + React Testing Library)
npm run postinstall # Runs patch-package automatically after install
npm run update-events  # Run script: tsx src/scripts/updateEventsSportType.ts
npm run update-status  # Run script: tsx src/scripts/updateEventStatus.ts
```

**Node requirement:** >=18.0.0

## Project Structure

```
teamup/
├── src/
│   ├── api/              # API webhook handlers
│   ├── assets/           # Static assets (avatars, images)
│   ├── components/       # Reusable UI components (flat structure, ~40 files)
│   │   └── ui/           # Primitive UI components (button, card, input, etc. - shadcn-style)
│   ├── constants/        # Static data (locations.ts with venue coords, avatars.ts)
│   ├── contexts/         # React Context providers
│   │   ├── SupabaseAuthContext.tsx  # Supabase auth provider (NEW)
│   │   ├── AuthContext.tsx          # Legacy Firebase auth provider
│   │   ├── CookieContext.tsx
│   │   └── ThemeContext.tsx
│   ├── hooks/
│   │   ├── useSupabaseAuth.ts       # Supabase auth hook (NEW)
│   │   ├── useSupabaseEvents.ts     # Supabase events hook (NEW)
│   │   ├── useAuth.ts              # Legacy Firebase auth hook
│   │   ├── useEvents.ts            # Legacy Firebase events hook
│   │   └── ...
│   ├── lib/
│   │   ├── supabase.ts             # Supabase client init (NEW)
│   │   └── utils.ts                # cn() helper
│   ├── pages/            # Route-level page components (~12 pages)
│   ├── services/
│   │   ├── supabaseNotificationService.ts  # Supabase notifications (NEW)
│   │   ├── supabaseEmailService.ts         # Supabase email service (NEW)
│   │   └── ...                             # Legacy Firebase services
│   ├── types/
│   │   ├── supabase.ts             # Supabase DB types + aliases (NEW)
│   │   └── index.ts                # Legacy app types
│   ├── App.tsx           # Root component with routing
│   ├── firebase.ts       # Legacy Firebase client init
│   └── index.tsx         # Entry point
├── api/                  # Vercel serverless API routes (NEW)
│   └── send-email.ts     # SendGrid email endpoint
├── supabase/             # Supabase project config (NEW)
│   ├── config.toml
│   └── migrations/
│       └── 20250223000001_initial_schema.sql
├── functions/            # Legacy Firebase Cloud Functions
├── patches/              # patch-package patches for react/react-dom 18.2.0
├── public/               # Static public files
├── vercel.json           # Vercel deployment config (updated for API routes)
└── .github/workflows/    # CI/CD
```

## Migration Status: Firebase → Supabase

The project is migrating from Firebase to Supabase. Both systems coexist during the transition.

### What has been migrated
- **Database schema:** Full PostgreSQL migration in `supabase/migrations/` with RLS policies
- **Auth hooks:** `useSupabaseAuth.ts` replaces `useAuth.ts`
- **Auth context:** `SupabaseAuthContext.tsx` replaces `AuthContext.tsx`
- **Events hook:** `useSupabaseEvents.ts` replaces `useEvents.ts`
- **Notifications:** `supabaseNotificationService.ts` with Realtime subscriptions
- **Email service:** `supabaseEmailService.ts` + Vercel API route (`api/send-email.ts`)
- **Storage:** 3 Supabase storage buckets (avatars, event-covers, memory-images)
- **Types:** Full Supabase DB types in `src/types/supabase.ts`

### What still uses Firebase (to migrate)
- Page components directly importing from `firebase/firestore` (EventDetails, Home, Community, etc.)
- Component-level Firestore calls (CreateEventDialog, EditEventDialog, etc.)
- Firebase Storage upload calls in components
- `firebase.ts` and `firebaseAdmin.ts` client init files

### Migration pattern for remaining components
When migrating a component from Firebase to Supabase:
1. Replace `import { db } from '../firebase'` with `import { supabase } from '../lib/supabase'`
2. Replace `collection/doc/getDocs/getDoc` calls with `supabase.from('table').select()`
3. Replace `addDoc/setDoc/updateDoc` with `supabase.from('table').insert/update/upsert`
4. Replace `deleteDoc` with `supabase.from('table').delete()`
5. Replace `arrayUnion/arrayRemove` with separate junction table operations
6. Replace `onSnapshot` with Supabase Realtime channels
7. Replace `ref/uploadBytes/getDownloadURL` with `supabase.storage.from('bucket')`

## Supabase Database Schema

### Tables (maps from Firestore collections)
- **profiles** — User profiles (extends auth.users via trigger)
- **events** — Sports events
- **event_players** — Junction: players registered for events (was `events.players[]`)
- **match_results** — Match scores (was `events.matchResults`)
- **friends** — Bidirectional friend relationships (was `friends.friends[]`)
- **friend_requests** — Pending friend requests (was `friends/{id}/requests`)
- **notifications** — In-app notifications
- **saved_events** — Bookmarked events
- **memories** — Post-event photo memories
- **memory_likes** — Memory likes (was `memories.likes[]`)

### Key differences from Firestore
- Arrays like `players[]`, `likes[]`, `friends[]` are normalized into junction tables
- `match_results` is a separate table instead of nested in events
- Email verification is handled natively by Supabase Auth (no custom tokens)
- Profile creation is automatic via database trigger on `auth.users` insert

## Architecture & Key Patterns

### Authentication Flow (Supabase)
- Supabase Auth with email/password and Google OAuth (`src/hooks/useSupabaseAuth.ts`)
- Profile auto-created via PostgreSQL trigger on signup
- Email verification handled natively by Supabase Auth
- `SupabaseAuthProvider` wraps the app providing `user`, `session`, `login`, `register`, `signInWithGoogle`, `signOut`
- Protected routes redirect to `/login` when unauthenticated

### State Management
- React Context API (no Redux/Zustand)
- Custom hooks for data fetching (`useSupabaseEvents`, `useSupabaseAuth`, `useCookies`)

### Routing (src/App.tsx)
```
/                    → Home (authenticated) or LandingPage (guest)
/login               → Login page
/register            → Registration page
/verify-email        → Email verification
/event/:id           → Event details (protected)
/my-events           → User's own events (protected)
/notifications       → Notifications view (protected)
/community           → Community page (protected)
/saved-events        → Saved/bookmarked events (protected)
/locations           → Venue locations list (protected)
/location/:locationId → Single venue details (protected)
```

### Email System
- **SendGrid via Vercel API route** (`api/send-email.ts`) — all transactional emails
- Client-side service in `src/services/supabaseEmailService.ts` calls the API route

### Styling Conventions
- **Primary accent color:** `#C1FF2F` (lime green)
- **Background:** `#111111` / `#1E1E1E` (dark theme default)
- Tailwind CSS for layout and utility classes
- MUI components for complex UI elements (dialogs, inputs)
- CSS variables via HSL for theming (shadcn/ui pattern in `src/index.css`)
- `cn()` utility from `src/lib/utils.ts` for conditional class merging (clsx + tailwind-merge)
- Mobile-first responsive design with 768px breakpoint
- Bottom navigation on mobile, top navbar on desktop

### Component Conventions
- Functional components with TypeScript (`React.FC`)
- Named exports preferred (some default exports exist)
- Components are flat in `src/components/` (no deep nesting)
- Primitive UI components in `src/components/ui/` follow shadcn/ui patterns
- Pages in `src/pages/` correspond to routes

## Environment Variables

```
# Supabase (required)
REACT_APP_SUPABASE_URL=https://your-project-id.supabase.co
REACT_APP_SUPABASE_ANON_KEY=your_supabase_anon_key

# SendGrid (server-side, for Vercel API route)
SENDGRID_API_KEY=your_sendgrid_api_key
SENDER_EMAIL=info@weteamup.app

# Optional
REACT_APP_MAILERLITE_API_KEY=your_mailerlite_api_key
REACT_APP_API_URL=              # Leave empty for Vercel (uses /api)
```

## Deployment

### Vercel (primary)
- Static build via `@vercel/static-build` (outputs `build/` dir)
- Serverless API routes in `api/` directory via `@vercel/node`
- Config in `vercel.json` — SPA fallback + API routing

### Legacy deployments (being phased out)
- Firebase Hosting (`.github/workflows/firebase-hosting-deploy.yml`)
- VPS via SCP + Nginx + PM2 (`.github/workflows/deploy.yml`)

## Testing

- **Framework:** Jest + React Testing Library
- **Config:** ESLint extends `react-app` and `react-app/jest`
- **Test files:** Co-located with source (`*.test.tsx`)
- **Run:** `npm test`
- Existing tests: `App.test.tsx`, `CreateEventDialog.test.tsx`, `Home.test.tsx`

## Important Notes

- The project uses `patch-package` with patches for React and ReactDOM 18.2.0 (in `patches/`)
- CRACO overrides webpack config for Node.js polyfill fallbacks (crypto, stream, path, etc.)
- Hot module replacement is disabled in dev (`craco.config.js`: `devServer.hot: false`)
- Locations are hardcoded in `src/constants/locations.ts` (Vilnius padel venues)
- Avatar system uses pre-defined avatar images (Avatar1-4) rather than user uploads
- The project name in `package.json` is "webpadel" but branding is "TeamUp" / "We Team Up"
- During migration, both Firebase and Supabase code coexist; prefer the `supabase`/`useSupabase*` versions for new work

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dzekuza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
