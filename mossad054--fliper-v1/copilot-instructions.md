## fliper-v1

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SwimEase is a React Native mobile app for swimming instruction and water skills learning. The app provides personalized swimming lessons, dryland exercises, progress tracking, AI-powered feedback, and gamification features to help users learn to swim at their own pace.

## Tech Stack

- **Framework**: React Native with Expo SDK 53
- **Language**: TypeScript (strict mode enabled)
- **Routing**: Expo Router (file-based routing with typed routes)
- **Styling**: NativeWind (Tailwind CSS for React Native)
- **State Management**: React Query (@tanstack/react-query), Zustand, React Context
- **Backend**: Supabase (PostgreSQL database with Row Level Security)
- **Icons**: Lucide React Native

## Development Commands

```bash
# Start Expo development server
npm start

# Run on specific platforms
npm run android
npm run ios
npm run web

# Linting
npm run lint
```

## Architecture

### Routing Structure

The app uses Expo Router's file-based routing system:

- **Root Layout** (`app/_layout.tsx`): Wraps the app with QueryClientProvider, ThemeProvider, and GestureHandlerRootView
- **Onboarding Flow**: splash → welcome → account → assessment → results
- **Tab Navigation** (`app/(tabs)/_layout.tsx`): 5 main tabs (Home, Lessons, Analytics, Calendar, More)
- **Dynamic Routes**:
  - `app/lessons/[id].tsx` - Individual lesson details
  - `app/dryland/[id].tsx` - Individual dryland exercise details

### Directory Structure

- **`/app`**: Expo Router pages and screens
  - `/(tabs)/`: Tab-based navigation screens
  - `/onboarding/`: User onboarding flow
  - `/lessons/[id].tsx`: Dynamic lesson detail pages
  - `/dryland/[id].tsx`: Dynamic dryland exercise pages
  - `/settings/`: Settings screens (accessibility, safety, support)
- **`/components`**: Reusable React components (LessonDetailModal, AIFeedbackCard, JournalModal, FloatingJournalButton)
- **`/contexts`**: React contexts (ThemeContext for light/dark theme management)
- **`/utils`**: Utility functions and API clients
  - `supabase.ts`: Supabase client and typed database functions
  - `aiFeedback.ts`: AI-powered feedback generation
  - `adaptiveLearning.ts`: Adaptive learning algorithm
  - `scheduledLessons.ts`: Lesson scheduling utilities
  - `dataSync.ts`: Data synchronization helpers
  - `themeUtils.ts`: Theme-related utilities
- **`/constants`**: Type definitions, colors, mock data
  - `types.ts`: Core TypeScript type definitions
  - `supabaseTypes.ts`: Auto-generated Supabase database types
  - `colors.ts`: Color palette constants
  - `mockData.ts`: Mock data for development
- **`/supabase`**: Database schema and migrations
  - `schema.sql`: Complete database schema with RLS policies

### Supabase Database Schema

The app uses the following main tables:

- **`user_profiles`**: User information, skill levels, progress, streaks, XP
- **`assessment_data`**: Initial user assessment responses
- **`journals`**: User journal entries (text, voice, video with AI analysis)
- **`lesson_feedback`**: User feedback on lessons (difficulty ratings, comments)
- **`scheduled_lessons`**: Calendar-scheduled lessons with reminders
- **`safety_checkins`**: Pre-swim safety checklist completions
- **`achievements`**: Unlockable achievements and badges
- **`leaderboard`**: XP rankings (weekly, monthly, all-time)
- **`skill_progress`**: Individual skill mastery tracking

All tables have Row Level Security (RLS) enabled with user-scoped policies.

### Theme System

The app uses a custom theme system managed by `ThemeContext`:

- Supports light and dark modes
- Theme preference persisted to AsyncStorage
- Access theme via `useTheme()` hook
- Theme colors organized into: primary, text, background, accent, ui categories
- Themes defined in `contexts/ThemeContext.tsx`

### TypeScript Configuration

- Path alias: `@/*` maps to project root (use `@/components`, `@/utils`, etc.)
- Strict mode enabled
- Expo TypeScript base configuration

## Key Patterns

### Supabase Integration

All Supabase operations are centralized in `utils/supabase.ts`:

```typescript
import { getUserProfile, updateUserProfile, createJournal } from '@/utils/supabase';

// Functions are fully typed using Database types from constants/supabaseTypes.ts
const profile = await getUserProfile(userId);
await updateUserProfile(userId, { streak: profile.streak + 1 });
```

### Theme Usage

```typescript
import { useTheme } from '@/contexts/ThemeContext';

function MyComponent() {
  const { theme, toggleTheme } = useTheme();
  return <View style={{ backgroundColor: theme.colors.background.white }} />;
}
```

### React Query

Data fetching uses React Query for caching, background updates, and optimistic updates:

```typescript
import { useQuery, useMutation } from '@tanstack/react-query';
```

### Navigation

Use Expo Router's typed navigation:

```typescript
import { router } from 'expo-router';

// Navigate to dynamic routes
router.push(`/lessons/${lessonId}`);
```

## Environment Variables

Required environment variables (in `.env` file):

- `EXPO_PUBLIC_SUPABASE_URL`: Supabase project URL
- `EXPO_PUBLIC_SUPABASE_ANON_KEY`: Supabase anonymous key

## Important Notes

- The app uses Expo's "new architecture" (`newArchEnabled: true` in app.json)
- Audio recording permissions are configured for both iOS and Android
- Media files (journals) are stored in Supabase Storage bucket `journal-media`
- The app supports iOS, Android, and web platforms

---
> Source: [Mossad054/fliper-v1](https://github.com/Mossad054/fliper-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
