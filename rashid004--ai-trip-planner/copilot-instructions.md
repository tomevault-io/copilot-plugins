## ai-trip-planner

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (Vite)
npm run build      # Type-check + build for production (tsc -b && vite build)
npm run lint       # Run ESLint
npm run preview    # Preview production build
```

There are no tests in this project.

## Environment Variables

Create a `.env` file at the root with:

```
VITE_GEMINI_API_KEY=        # Google Gemini API key
VITE_GOOGLE_PLACE_API_KEY=  # Google Places API key (for photo fetching)
```

Firebase config is hardcoded in [src/lib/firebase.ts](src/lib/firebase.ts).

## Architecture Overview

This is a React + TypeScript + Vite SPA — an AI-powered trip planner.

### Data Flow

1. User fills [CreateTrip](src/components/Trip/CreateTrip.tsx) form (destination via Photon geocoding, duration, budget, travel group)
2. On submit, [generateTripPlan](src/service/gemini.tsx) calls Gemini 2.0 Flash with the prompt template from [src/const/option.ts](src/const/option.ts), returning structured JSON
3. The trip is saved to Firebase Firestore (`AiTrips` collection) via [supabaseTripService.ts](src/service/supabaseTripService.ts) (misleading name — it's actually Firebase, not Supabase)
4. User is redirected to `/view-trip/:tripId` which fetches and renders the saved trip

### Key Modules

- **Auth** — Google Sign-In via Firebase Auth ([src/service/auth.ts](src/service/auth.ts)). User object is persisted to `localStorage` and read back via the `useLocalUser` hook ([src/hooks/useLocalUser.ts](src/hooks/useLocalUser.ts)). `App.tsx` subscribes to `onAuthStateChanged` to keep localStorage in sync.
- **AI** — [src/service/gemini.tsx](src/service/gemini.tsx) wraps `@google/generative-ai`. The prompt is in `AI_PROMPT` in [src/const/option.ts](src/const/option.ts). The response is JSON (enforced via `responseMimeType: "application/json"`).
- **Firestore** — [src/service/supabaseTripService.ts](src/service/supabaseTripService.ts) handles all Firestore CRUD (get all trips by user email, get by ID, create). Collection name: `AiTrips`.
- **Place photos** — [src/service/globalApi.tsx](src/service/globalApi.tsx) calls Google Places API v1 (`places:searchText`) to fetch place photos displayed in trip views.
- **Location autocomplete** — [PhotonAutocomplete](src/components/common/PhotonAutocomplete.tsx) queries the free Photon/Komoot geocoding API (no API key required).

### Route Structure

| Path | Component |
|---|---|
| `/` | `Home` (landing page) |
| `/create-trip` | `CreateTrip` (trip generation form) |
| `/view-trip/:tripId` | `ViewTrip` (trip detail with hotels + itinerary) |
| `/my-trip` | `MyTrips` (user's saved trips list, requires auth) |

### Type Definitions

Core types are in [src/types/trip.ts](src/types/trip.ts). `TripRecord` is the Firestore document shape. The Gemini response is expected to match `TripRecord.tripData.travelPlan` (hotels array + itinerary array).

### Styling

Tailwind CSS v4 (via `@tailwindcss/vite` plugin). Custom utility classes (`section-container`, `card-box`, `label-title`, etc.) are defined in [src/index.css](src/index.css). Path alias `@` maps to `src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rashid004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
