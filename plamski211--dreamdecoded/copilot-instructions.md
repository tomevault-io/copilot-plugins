## dreamdecoded

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npx expo start` -- start dev server (or `npm start`)
- `npx expo start --ios` -- run on iOS simulator
- `npx expo start --android` -- run on Android emulator
- `npx expo start --web` -- run in browser
- `npx expo export --platform web` -- production web build (outputs to `dist/`)
- No test runner or linter is configured.

## Architecture

React Native dream journaling app built with **Expo SDK 54** (new arch enabled), **expo-router v6** for file-based routing, **Zustand** for state management. Dreams are recorded via microphone, sent to **Google Gemini 2.5-flash** for analysis (transcription, mood extraction, symbol identification, interpretation), and persisted locally. Key deps: `date-fns` for date formatting, `lucide-react-native` for icons.

### Routing (`app/`)

expo-router file-based routing with three route groups:

- `(auth)/` -- welcome, sign-in, sign-up screens (Stack layout)
- `(tabs)/` -- main app tabs: Home (recording), Journal, Insights, Profile
- `dream/[id]` -- dream detail; `dream/[id]/ask` -- conversational "Ask Your Dream" modal
- `onboarding/` -- first-run onboarding flow
- `paywall` -- presented as a modal (slide from bottom)

Auth guard in root `_layout.tsx` (`useProtectedRoute`) redirects based on session/onboarding state. Uses `onAuthStateChange` with `INITIAL_SESSION` event (not `getSession()`) to avoid race conditions. App works offline-first with a default local user (no auth required).

### State (`lib/store.ts`)

Single Zustand store (`useAppStore`) holds auth state, recording state, processing state, dream list, subscription status, and current decoded dream. Access slices via selectors: `useAppStore((s) => s.fieldName)`. Store actions persist dreams to local storage **and** call `upsertDream`/`deleteDreamRemote` to Supabase as a side effect (local-first, cloud sync optional).

### AI Integration (`lib/ai.ts`)

`processDream()` routes audio to the `process-dream` Supabase edge function, which calls Gemini 2.5-flash. Returns JSON with title, transcription, summary, moods (with confidence scores), symbols (with meanings), and interpretation. `askDream()` routes through the `ask-dream` edge function for multi-turn conversational follow-up, passing dream context (title, transcription, summary, interpretation). `friendlyError()` sanitizes error messages to strip API keys and quota info before showing to users.

API key loaded from `EXPO_PUBLIC_GEMINI_API_KEY` env var.

### Supabase Backend (`supabase/`)

Six Deno edge functions handle all AI and analytics:

- `process-dream` -- transcription + mood/symbol extraction via Gemini
- `ask-dream` -- multi-turn dream conversation
- `analyze-dream` -- pattern detection (mood shifts, recurring symbols)
- `transcribe-dream` -- speech-to-text preprocessing
- `generate-report` -- weekly summary generation (called from Insights tab)
- `generate-dream-art` -- AI art generation (placeholder)

All functions include CORS headers, Gemini schema validation (`response_mime_type`), and robust JSON parsing (direct parse → markdown strip → brace extraction fallback). Database: `profiles` and `dreams` tables with RLS; auto-create profile trigger on signup.

### Persistence (`lib/storage.ts`, `lib/storage.web.ts`)

Platform-split persistence: `expo-sqlite` on native (tables: `dreams`, `preferences`), `localStorage` on web. Same API surface: `initDB()`, `saveDream()`, `loadDreams()`, `deleteDream()`, `savePreference()`, `loadPreference()`, `loadAllPreferences()`. Metro resolves `storage.web.ts` automatically on web.

### Audio Recording (`lib/audio.ts`)

Cross-platform: `expo-av` with HIGH_QUALITY preset + dB metering on native, `MediaRecorder` + Web Audio API frequency analysis on web. Real-time audio level callbacks (normalized 0-1) drive the breathing orb animation. Recording auto-stops at 600 seconds (10-minute limit). `lib/speechRecognition.ts` provides a Web Speech API wrapper (web-only) with interim transcript tracking and auto-restart on silence.

### Design System (`lib/theme.ts`, `lib/ThemeContext.tsx`)

Multi-theme system with 4 themes:

| Theme | Background | Mood | Fonts |
|-------|-----------|------|-------|
| **VOID** (default) | True black `#000` | Monochrome, minimal | DMSans |
| **INK** | Warm cream `#F5F0E8` | Serif, journal-like | EBGaramond (italic) |
| **DUSK** | Charcoal `#1C1B18` | Sage green accents | DMSans |
| **FROST** | Blue-gray `#F8F9FC` | Frosted, Scandinavian | DMSans |

`lib/theme.ts` exports the `themes` record, `ThemeKey` type, and **layout-only** tokens (`spacing`, `radius`, `fontSize`, `lineHeight`, `SCREEN_PADDING`). Each theme has 28 color tokens, font families, orb animation config, `headingStyle`, and `labelStyle`.

`lib/ThemeContext.tsx` provides `ThemeProvider`, `useTheme()`, and `useColors()` hooks. Theme persists via `expo-secure-store` (native) / `localStorage` (web).

**Styling rule**: `StyleSheet.create` for layout only. Colors and fonts always inline:
```tsx
const { theme } = useTheme();
const c = theme.colors;
<Text style={[styles.title, { color: c.text, fontFamily: theme.fonts.heading }]}>
```
Zero hardcoded colors or font families outside `theme.ts`. No `LinearGradient` — all backgrounds are solid colors. Mood gradients (fixed across themes) are the only gradient colors, used in `MoodBadge`.

### Types (`types/index.ts`)

Core domain types: `User`, `Dream`, `MoodTag`, `DreamSymbol`, `DreamConversation`, `ConversationMessage`, `WeeklyReport`, `PatternAlert`. `MoodName` union: `'peaceful' | 'anxious' | 'joyful' | 'confused' | 'sad' | 'excited' | 'fearful' | 'neutral'`.

## Key Patterns

- **Path aliases**: `@/` maps to project root via tsconfig paths. Use `@/lib/*`, `@/components/*`, `@/types/*`.
- **Web compatibility**: `Platform.OS === 'web'` checks throughout. Custom `AlertModal` component replaces `Alert.alert()` (which fails on web with 3+ buttons). `lib/storage.web.ts` replaces SQLite. `lib/notifications.ts` returns no-ops on web. `lib/speechRecognition.ts` is web-only.
- **Animations**: React Native's built-in `Animated` API (no native dependencies). `FadeInView` component provides staggered fade-in-down entrance animations. `GlowOrb` uses `Animated.loop`/`Animated.sequence` for breathing scale + audio level reactivity. `CalendarStrip` uses `LayoutAnimation` on Android for month/week toggle.
- **Haptic feedback**: `lib/haptics.ts` wraps `expo-haptics` — `lightTap`, `mediumTap`, `success`, etc.
- **Mood system**: 8 moods with emoji and gradient pair defined in `lib/dreamAnalysis.ts` via `MOOD_CONFIG`. Gradients are fixed across themes (they represent emotions, not theme identity).
- **Google Auth**: `lib/googleAuth.ts` provides `useGoogleAuth()` hook via `expo-auth-session` and `signInWithGoogleIdToken()` for Supabase integration. Used in sign-in/sign-up screens.
- **Subscription**: `isPremium` defaults to `true`. `lib/subscription.ts` has RevenueCat stubs (not wired).
- **Supabase**: Optional. `lib/supabase.ts` checks `hasCredentials` before making calls. App functions fully offline.
- **Web deployment**: Deployed to Vercel via `vercel.json` — builds with `npx expo export --platform web`, outputs `dist/`, all routes rewrite to `/index.html` (SPA mode).

## Environment Variables

```
EXPO_PUBLIC_GEMINI_API_KEY=...       # Required: Google AI Studio API key
EXPO_PUBLIC_SUPABASE_URL=...         # Optional: Supabase project URL
EXPO_PUBLIC_SUPABASE_ANON_KEY=...    # Optional: Supabase anon key
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/plamski211) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
