## supernova-travel

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
npx expo start          # start Metro bundler (press i/a/w for iOS/Android/Web)
npx expo start --ios    # launch iOS simulator directly
npx expo start --android
npm run lint            # expo lint (ESLint)
npm test                # jest --watchAll
cd functions && npm run build   # compile Cloud Functions TypeScript
cd functions && npm run deploy  # deploy Cloud Functions to Firebase
```

There is no separate build step for local dev — Expo handles transpilation at runtime. Cloud builds use EAS (`eas build --profile development|preview|production`).

**EAS build profiles** (`eas.json`):
- `development` — dev client; iOS simulator + Android APK
- `preview` — internal distribution (device install)
- `production` — app store submission with auto-incremented versions

## Environment

Copy `.env.local.example` to `.env.local` and fill in all keys. Client vars are prefixed `EXPO_PUBLIC_` so Expo exposes them to the bundle. Cloud Function vars are set via `firebase functions:config:set` or Firebase environment secrets — never `EXPO_PUBLIC_`.

| Variable | Where | Purpose |
|---|---|---|
| `EXPO_PUBLIC_FIREBASE_API_KEY` | client | Firebase API key |
| `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN` | client | Firebase auth domain |
| `EXPO_PUBLIC_FIREBASE_PROJECT_ID` | client | Firebase project ID |
| `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET` | client | Firebase storage bucket |
| `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | client | Firebase messaging sender ID |
| `EXPO_PUBLIC_FIREBASE_APP_ID` | client | Firebase app ID |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | client | Google Places autocomplete (New API) |
| `EXPO_PUBLIC_REVENUECAT_IOS_KEY` | client | RevenueCat iOS SDK |
| `EXPO_PUBLIC_REVENUECAT_ANDROID_KEY` | client | RevenueCat Android SDK |
| `EXPO_PUBLIC_ALGOLIA_APP_ID` | client | Algolia search app ID |
| `EXPO_PUBLIC_ALGOLIA_SEARCH_KEY` | client | Algolia **Search-Only** key (never Admin) |
| `ALGOLIA_APP_ID` | Cloud Function | Algolia sync (Admin key context) |
| `ALGOLIA_ADMIN_KEY` | Cloud Function | Algolia index write access |
| `AVIATIONSTACK_API_KEY` | Cloud Function | Flight status polling |
| `GEMINI_API_KEY` | Cloud Function | AI trip generation (never expose to client) |

## Architecture

### Routing (Expo Router v6)

File-based routing under `app/`. App bundle ID: `com.supernovatravel.app`. React Native New Architecture is enabled.

```
app/
├── _layout.tsx                    # Root layout — auth listener, RevenueCat init, push token
├── (auth)/
│   ├── _layout.tsx
│   ├── welcome.tsx
│   ├── sign-in.tsx
│   ├── sign-up.tsx
│   └── forgot-password.tsx
├── (tabs)/
│   ├── _layout.tsx                # Tab bar: Feed, Explore, Create, Search, Profile
│   ├── index.tsx                  # Feed
│   ├── explore.tsx
│   ├── create.tsx
│   ├── search.tsx
│   └── profile.tsx
├── (wallet)/
│   ├── _layout.tsx
│   ├── boarding-passes.tsx        # List
│   ├── boarding-pass/[id].tsx     # Detail
│   ├── boarding-pass/add.tsx      # Add form
│   ├── reservations.tsx           # List
│   ├── reservation/[id].tsx       # Detail
│   ├── loyalty.tsx                # List
│   ├── loyalty/[id].tsx           # Detail
│   └── loyalty/add.tsx            # Add form
├── trip/
│   ├── [id].tsx                   # Trip detail (modal)
│   ├── new.tsx                    # Manual trip wizard (modal)
│   ├── ai-generate.tsx            # AI generation form (modal)
│   └── ai-generating.tsx          # Generation loading screen → routes to trip/[id]
├── post/[id].tsx                  # Post detail (modal)
├── user/[uid].tsx                 # Public profile (full-screen push)
├── settings.tsx                   # Theme toggle, account, sign out (modal)
└── paywall.tsx                    # RevenueCat paywall (modal)
```

**Auth routing is centralised in `app/_layout.tsx`** via a single `onAuthStateChanged` listener. On sign-in it calls `configureRevenueCat(uid)`, registers the Expo push token, fetches the user's `tier`, then `router.replace('/(tabs)')`. On sign-out: `router.replace('/(auth)/welcome')`. There is no route guard middleware.

Path alias `@/` maps to the project root (see `tsconfig.json`).

### State Management

Three Zustand stores in `stores/`:
- `useAuthStore` — Firebase `User | null`, `tier: 'free' | 'pro' | 'business'`, initialization flag
- `useUserStore` — cached `UserProfile` (displayName, avatarUrl, bio, location, follower counts)
- `useThemeStore` — `mode: 'dark' | 'light' | 'system'`, persisted via AsyncStorage; `useTheme()` hook resolves `'system'` via `useColorScheme`

TanStack React Query (staleTime 2 min, 2 retries) wraps all Firestore reads. **No `onSnapshot` listeners in hooks** — use `getDocs`/`getDoc` only. Exception: post comments screen uses `onSnapshot` directly in a `useEffect`.

### Firebase

`services/firebase.ts` exports `auth`, `db`, `storage`, `functions` as named singletons (region: `us-central1`). Import these directly — never call `getAuth()` / `getFirestore()` elsewhere.

Firestore collections:
- `users/{uid}` — profile + `tier` + `expoPushTokens[]`; subcollections: `feed/`, `notifications/`, `savedTrips/`
- `posts/{postId}` — travel posts; subcollection `comments/`
- `trips/{tripId}` — itineraries; subcollections: `days/{dayId}`, `days/{dayId}/activities/{activityId}`
- `follows/{docId}` — follow graph
- `boarding_passes/{passId}` — owner-only (`isOwner(resource.data.ownerUid)`)
- `reservations/{reservationId}` — owner-only
- `loyalty_programs/{programId}` — owner-only
- `usage_quotas/{uid}` — server-side Admin SDK only
- `reports/{reportId}` — create-only from client

**The `feed/` and `notifications/` subcollections are write-only from Cloud Functions.** `usage_quotas` is entirely server-side.

**Firestore security rules** (`firestore.rules`): public trips/posts are readable by all; boarding passes, reservations, and loyalty programs are owner-only; `usage_quotas` has no client access at all; `reports` is create-only from the client.

### Cloud Functions (`functions/src/`)

All functions use Firebase Functions v2.

- `generateTrip` (`generateTrip.ts`) — HTTPS callable; receives `GenerateTripRequest`, calls Gemini 1.5 Flash, writes `trips` + `days` + `activities` subcollections, enforces weekly quota via `usage_quotas`
- `checkFlightStatus` (`checkFlightStatus.ts`) — Cloud Scheduler every 30 min; queries upcoming boarding passes, calls AviationStack HTTP API, updates Firestore status, sends Expo push notifications
- `syncTripToAlgolia` / `syncUserToAlgolia` (`syncAlgolia.ts`) — `onDocumentWritten` triggers; upserts/deletes public trips in the Algolia `trips` index and users in the `users` index
- `types.ts` — shared TypeScript interfaces for Cloud Function request/response shapes

**Never call Gemini or any third-party secret API directly from client code.** All such calls go through Cloud Functions.

### Services

- `services/firebase.ts` — `auth`, `db`, `storage`, `functions` singletons
- `services/revenuecat.ts` — `configureRevenueCat(uid)`: sets log level, calls `Purchases.configure` with platform-specific keys; called in `_layout.tsx` after auth fires
- `services/gemini.ts` — `callGenerateTrip(request)`: calls the `generateTrip` Cloud Function via `httpsCallable`

### Hooks (`hooks/`)

| Hook | Returns |
|---|---|
| `useTheme` | `{ colors, isDark, mode }` — resolves system theme |
| `useFeed` | Infinite-paginated personalized feed posts (TanStack Query) |
| `usePost(id)` | Single post query by ID |
| `useSearch(text)` | `{ users, trips, isSearching }` — Algolia v5, 350ms debounce |
| `useExplore` | `{ trips, tripsLoading, suggestions, suggestionsLoading }` |
| `usePublicProfile(uid)` | `{ profile, isLoading, isFollowing, isOwnProfile }` |
| `useUserProfile(uid)` | Raw user profile query by UID |
| `useFollow(uid)` | `{ follow, unfollow }` mutations |
| `useTripList(uid)` | TanStack Query result for user's trips (also exports `usePublicTrips`) |
| `useTrip(id)` | Single trip query with nested days/activities |
| `useCreateTrip` | Create trip mutation |
| `useAiGenerateTrip` | AI generation mutation; redirects to `/paywall` on quota exceeded |
| `useBoardingPasses` | `{ boardingPasses, isLoading, addPass, deletePass }` |
| `useReservations` | `{ reservations, isLoading, addReservation, deleteReservation }` |
| `useLoyaltyPrograms` | `{ loyaltyPrograms, isLoading, addProgram, deleteProgram }` |
| `usePurchases` | `{ purchasePro, restorePurchases, isLoading, error }` |

### Tier / Monetisation

The tier (`free | pro | business`) is fetched from Firestore `users/{uid}.tier` on every auth state change and stored in `useAuthStore`. RevenueCat (`react-native-purchases`) handles purchase flows — `usePurchases` wraps `Purchases.purchasePackage` and `Purchases.restorePurchases`. A `syncTier` Cloud Function (webhook) is expected to update `users/{uid}.tier` after a successful purchase; `useAuthStore.tier` is the authoritative runtime source.

Free tier: 1 AI-generated trip per week (enforced server-side via `usage_quotas`).

### TypeScript Types (`types/`)

`types/index.ts` — all core domain types:

| Type/Interface | Description |
|---|---|
| `Tier` | `'free' \| 'pro' \| 'business'` |
| `ThemeMode` | `'dark' \| 'light' \| 'system'` |
| `UserProfile` | uid, displayName, username, avatarUrl, bio, location, follower/following/tripsCount, tier, createdAt |
| `Post` | travel post with authorUid, caption, mediaType, mediaUrl, placeName/placeId/lat/lng, likesCount, commentsCount, tags |
| `Comment` | authorUid, text, createdAt |
| `Trip` | title, destination (name/placeId/lat/lng/countryCode), visibility, collaborators[], isAiGenerated, status, tags, likesCount, savesCount |
| `TripWithDays` | `Trip` extended with `days: TripDay[]` |
| `TripDay` | dayNumber, date, title, notes, activities[] (loaded from subcollection client-side) |
| `TripActivity` | type (ActivityType), title, placeId, startTime/endTime (wall-clock strings, NOT Timestamps), durationMinutes, notes, bookingRef, cost, currency, mediaUrls, order |
| `ActivityType` | `'flight' \| 'hotel' \| 'restaurant' \| 'activity' \| 'transport' \| 'free'` |
| `TripStatus` | `'planning' \| 'active' \| 'completed'` |
| `TripVisibility` | `'public' \| 'followers' \| 'private'` |
| `BoardingPass` | airline, flightNumber, origin/destination (IATA codes), departureTime/arrivalTime (ISO 8601), seat, gate, barcode, status |
| `BoardingPassStatus` | `'upcoming' \| 'checked_in' \| 'boarded' \| 'completed' \| 'cancelled'` |
| `Reservation` | type (ReservationType), title, confirmationCode, checkIn/checkOut (ISO 8601 date) |
| `ReservationType` | `'hotel' \| 'airbnb' \| 'rental_car' \| 'restaurant' \| 'activity'` |
| `LoyaltyProgram` | programType, programName, memberNumber, balance, unit, tier, expiryDate, isManual |
| `LoyaltyUnit` | `'miles' \| 'points' \| 'nights' \| 'segments'` |
| `LoyaltyTier` | `'standard' \| 'silver' \| 'gold' \| 'platinum' \| 'diamond'` |
| `CreateTripInput` / `UpdateTripInput` | mutation input shapes |

`types/ai.ts` — AI generation types:
- `TravelStyle`: `'adventure' | 'luxury' | 'budget' | 'family' | 'cultural'`
- `GenerateTripRequest`: destination, countryCode, startDate/endDate (ISO string | null), durationDays, travelStyle, mustSee[], preferences

### Design System

All design tokens live in `constants/`:
- `colors.ts` — exports `DarkColors` and `LightColors`; `useTheme()` resolves the correct set. Brand: purple `#a78bfa`, pink `#f472b6`, blue `#60a5fa`. Accent: amber `#fbbf24`, teal `#34d399`. `colors.text.inverse` = text colour for branded (purple) surfaces.
- `typography.ts` — `FontSize`, `FontWeight`, `FontFamily`, `LineHeight`, `LetterSpacing`
- `spacing.ts` — `Spacing` (4px base), `BorderRadius`, `Shadow` (`Shadow.sm / .md / .lg` — purple-tinted)
- `icons.ts` — Phosphor icon + semantic color maps; import from here instead of hard-coding icon/color pairs:
  - `ACTIVITY_ICONS: Record<ActivityType, { Icon, color }>` — blue flights, purple hotels, pink restaurants, teal activities, amber transport, grey free time
  - `RESERVATION_ICONS: Record<ReservationType, { Icon, color }>`
  - `LOYALTY_ICONS: Record<LoyaltyProgram['programType'], { Icon, color }>`
  - `VISIBILITY_ICONS: Record<TripVisibility, { Icon, color }>`
  - `PAYWALL_FEATURE_ICONS: Array<{ Icon, color, label, description }>` — 6 pro-tier features for paywall screen
  - `TAB_ICONS: Record<string, PhosphorIcon>` — tab bar icons (Create tab uses a gradient `+` circle, not an icon)
  - `PhosphorIcon` — re-exported `Icon` type from `phosphor-react-native`

UI primitives in `components/ui/`:
- `Button` — variants: `primary` (LinearGradient purple→pink), `secondary`, `ghost`, `danger`; sizes: `sm | md | lg`; props: `label`, `onPress`, `loading?`, `disabled?`
- `GlassCard` — `BlurView` frosted glass with configurable `intensity`
- `Avatar` — sizes: `xs | sm | md | lg | xl`; props: `uri?`, `name`, `size`
- `Badge` — pill/tag component
- `TypeIconBubble` — 44×24 icon bubble for activity/reservation type; uses `ACTIVITY_ICONS`/`RESERVATION_ICONS` maps

Feed components in `components/feed/`:
- `FeedCard` — travel post card (photo/video + author metadata, like/comment counts)
- `FeedActions` — like, comment, and save buttons row
- `VideoPlayer` — video playback; uses `expo-av`

Trip components in `components/trip/`:
- `TripCard` — trip preview card for grids and lists
- `DayTimeline` — day-by-day itinerary timeline visualization
- `ActivityItem` — individual activity row (uses `ACTIVITY_ICONS` for type icon + color)
- `AiPromptForm` — AI generation form: destination, dates, travel style (`TravelStyle`), must-see, free-text preferences
- `AiGeneratingAnimation` — loading animation during AI generation (always-dark, does not use `useTheme`)

Wallet components in `components/wallet/`:
- `BoardingPassCard` — always-dark physical boarding pass card (`#1a1035 → #0f0a2a` gradient)
- `BarcodeDisplay` — QR code via `react-native-qrcode-svg` on white background (scanner constraint)
- `ReservationCard` — list row with `TypeIconBubble` (44×24), title, confirmation code, check-in date
- `LoyaltyCard` — full-width card with `PointsBalance` and inline Phosphor type icon
- `PointsBalance` — formatted balance with `Intl.NumberFormat`, tier badge

Profile components in `components/profile/`:
- `EditProfileSheet` — RN `Modal` (`pageSheet`) for editing display name, bio, location
- `PostsGrid`, `TripsGrid`, `SavedGrid` — profile tab content (Posts/Saved are placeholders)

Search components in `components/search/`:
- `UserResult` — user search result item
- `TripResult` — trip search result item
- `PlaceResult` — Google Places search result item

Explore components in `components/explore/`:
- `UserSuggestion` — suggested user card
- `TrendingCard` — trending trip card
- `TripGrid` — grid layout for trending trips

Other components:
- `components/SplashOverlay` — overlay shown during app initialization (before auth resolves)
- `components/paywall/PaywallFeatureList` — pro tier features list; driven by `PAYWALL_FEATURE_ICONS`

**Platform handling**: iOS tab bar and translucent surfaces use `BlurView`; Android uses solid `rgba(10,10,26,0.95)`. Follow this pattern for any frosted-glass UI.

**List performance**: Use `@shopify/flash-list` (`FlashList`) instead of `FlatList` for all scrollable lists. `react-native-draggable-flatlist` is available for drag-to-reorder (e.g., trip activity ordering).

## UI/UX Design Philosophy

**Before writing ANY user-facing UI, read `.Codex/skills/supernova-design/SKILL.md` and run its
pre-ship checklist.** That skill is the authority; this section is the summary. UI shipped without
running the checklist is not done.

### The direction: light editorial

Supernova is a **light, warm, editorial travel app where photography is the hero.** Think a printed
travel magazine — generous whitespace, confident type, photos that breathe. Dark is reserved
exclusively for **immersive moments**: the Mapbox globe/trip map, the splash screen, and the
AI-generating screen. Those earn darkness; a settings list does not.

Rationale: dark-navy-with-a-purple-gradient is the default aesthetic of every AI-generated app. It's
forgiving — it hides bad spacing and weak type. Light is harder, which is why it reads as designed.
And travel photography needs whitespace the way a painting needs a gallery wall.

### Hard rules (violating any of these is a bug, not a preference)

1. **No emoji. Ever.** Phosphor icons only, from `constants/icons.ts` where a semantic map exists.
   Duotone for semantic/type icons; bold/regular for small utility icons (X, Plus, chevrons).
2. **No dashed borders on actions.** Dashed = dropzone semantics. It reads as a wireframe.
3. **One primary action per screen.** Everything else is secondary or a text link. Three
   identical-weight buttons means the screen has no hierarchy.
4. **Every empty state = icon + title + description + action.** "No activities yet" in grey is not
   an empty state. Name the space and invite: "Start your first day."
5. **Photos lead** on any place/trip/post screen.
6. **Motion on every state change** — house spring `tension: 65, friction: 11`.
7. **Haptics** — `Light` on nav/select, `Medium` on create/add/destructive.
8. **Touch targets ≥44pt; body contrast ≥4.5:1; icon buttons need `accessibilityLabel`.**

### Palette

**Light chrome (default — all app surfaces).** Warm neutrals, never clinical white:

| Token | Value | Use |
|---|---|---|
| Canvas | `#FBF9F5` | Page background |
| Surface | `#FFFFFF` | Cards, sheets |
| Sunken | `#F0EAE0` | Chips, inset areas |
| Hairline | `#E5DDD2` | Dividers (0.5px) |
| Text primary | `#1F1C19` | Headings, body |
| Text secondary | `#6B6157` | Supporting copy |
| Text muted | `#9A8F82` | Metadata, eyebrow labels |
| Text disabled | `#C4B8A8` | Empty-state icons |

**Primary action:** near-black `#1F1C19` with `#FBF9F5` text. Confident, not shouty. Do NOT make
every CTA a purple gradient — that's the AI-default tell.

**Brand accent — sparingly.** The star's violet→pink gradient is a *jewel against neutrals*, not
wallpaper. Reserve for: the star mark, active/selected states, and at most one hero CTA per flow
(e.g. "Generate with AI"). Purple `#7F77DD` · Pink `#D4537E`.

**Semantic:** keep `ACTIVITY_ICONS` (blue flights, purple hotels, pink restaurants, teal activities,
amber transport). On light, use the color for the icon and a ~10% tint for its bubble.

**Dark — immersive moments ONLY** (globe/trip map, splash, AI-generating): Void `#0B0A12` · Elevated
`#171422` · Hairline `#26232E` · Text `#F5F3F9` / `#9C95AD`. The light→dark transition is a
signature moment — fade/scale it, never hard-cut.

### Typography

| Role | Size | Weight | Notes |
|---|---|---|---|
| Screen title | 26–30 | 500–600 | Tight tracking (`-0.02em`) |
| Section head | 17 | 500 | |
| Body | 15 | 400 | `lineHeight: 1.5` |
| Eyebrow / meta | 11 | 500 | Uppercase, letterspaced `0.08em`, muted |
| Caption | 12–13 | 400 | Muted |

**The eyebrow label is the editorial signature** — small, tracked-out, muted, sitting above a big
title (`JUL 25 – 30 · 6 DAYS` above `Trip to Pacific Beach`). This single pattern does more for the
editorial feel than anything else. Sentence case everywhere; never Title Case buttons.

### Spacing & shape

Screen margins **≥20px** (generosity is the point). Card radius 16–20px; buttons/chips 12px or full
pill. Vertical rhythm 8/12/16/20/32. Hairlines `0.5px`. Align **optically**, not just
mathematically.

### Copy voice

Sentence case, contractions, verb-first. Buttons name the verb ("Find a place", not "Submit").
Empty states invite, never apologize. Errors say what happened and what to do. Skip
"successfully", "please", "simply", "just", and exclamation marks.

### Anti-patterns — the AI-generated tells

Dark navy + purple gradient on everything · decorative glassmorphism · gradient on every button ·
emoji as icons · dashed placeholder boxes shipped as real UI · three equal-weight buttons in a row ·
grey "No items yet" as an empty state · perfectly even spacing with no rhythm · cards with borders
AND shadows AND fills.

## Architecture Rules

These rules apply to ALL new code:

1. **No direct Gemini / secret API calls from client** — Cloud Function proxy only
2. **No `onSnapshot` in TanStack Query hooks** — use `getDocs`/`getDoc`. Exception: post comments
3. **All components use `const { colors } = useTheme()`** — never import `DarkColors`/`LightColors` directly, except in always-dark screens: the Mapbox globe/trip map (`app/(tabs)/search.tsx`, `components/trip/TripMapView`), the splash screen (`SplashOverlay`), the AI-generating screens (`AiGeneratingAnimation`, `app/trip/ai-generating.tsx`), and `BoardingPassCard` (kept dark deliberately — a boarding pass is a physical-object skeuomorph, not app chrome)
4. **`StyleSheet.create` is module-level** — it cannot call `useTheme()`. Dynamic/theme-dependent colors go in **inline styles only**, not inside `StyleSheet.create`
5. **`LinearGradient` colors prop** must be typed as `[string, string]`, not `string[]`
6. **`useCallback`** required for all event handlers passed as props to child components
7. **Import `auth`, `db`, `storage`, `functions` from `services/firebase`** — never call `getAuth()`/`getFirestore()` elsewhere
8. **Algolia Search-Only Key on client** — `EXPO_PUBLIC_ALGOLIA_SEARCH_KEY` is read-only. Admin key stays in Cloud Functions only
9. **`@/` path alias** for all imports (maps to project root via `tsconfig.json`)
10. **Haptics**: `expo-haptics` for tap feedback — `Light` on tab press, `Medium` on follow/create actions
11. **Icon + color pairs**: always pull from `constants/icons.ts` maps (`ACTIVITY_ICONS`, `RESERVATION_ICONS`, etc.) — never hard-code icon components or hex colors for typed entities inline
12. **Lists**: use `FlashList` from `@shopify/flash-list` — not `FlatList` — for all scrollable content lists
13. **`TripActivity.startTime` / `endTime`** are wall-clock strings (`"14:30"`), not Firestore Timestamps — never coerce them to Date objects

---
> Source: [markolsen619/supernova-travel](https://github.com/markolsen619/supernova-travel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
