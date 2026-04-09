## ai-expo-supabase-superwall-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI-first Expo mobile app built with TypeScript, designed for rapid development with AI assistance. The stack is optimized for lowest friction and single-language (TypeScript) development everywhere.

### Core Stack
- **App**: Expo (managed) + `expo-router` + TypeScript + EAS Build
- **Backend**: Supabase (Auth, Postgres, RLS, Storage, pgvector, Edge Functions in Deno/TS)
- **AI**: Supabase Edge Function proxy with SSE streaming; single endpoint abstracts providers
- **UI**: React Native Paper (MD3) only; no Tailwind/Nativewind to avoid class churn
- **Forms/Types**: React Hook Form + Zod; `zod` schemas = DB/API contract
- **Data fetching**: TanStack Query; minimal global state (Zustand for tiny UI bits only)
- **Analytics/Crash**: PostHog + Sentry (Expo plugin)
- **Paywall**: Superwall (Expo config plugin); entitlements gate premium screens
- **Notifications**: `expo-notifications` + Expo Push
- **Config**: `app.config.ts` + Zod-validated env; secrets only in Edge Functions
- **Offline**: TanStack Query cache + `react-native-mmkv` (skip complex offline writes in v1)

### Why These Choices
- **Single language (TS) everywhere** reduces ambiguity and back-and-forth
- **SQL + RLS** is easier to reason about access rules than Firestore rules
- **Typed Supabase** (`supabase gen types`) gives compile-time guidance
- **Paper** avoids styling bikeshedding; predictable components and theming
- **Edge Functions** keep API keys off-device; one place to add new models/providers
- **Expo managed** avoids native spelunking; fewer manual steps

## Common Development Commands

```bash
# Install dependencies
pnpm install

# Start development server
pnpm expo start

# Platform-specific development
pnpm run android  # Start Android emulator
pnpm run ios      # Start iOS simulator
pnpm run web      # Start web version

# Linting
pnpm run lint

# Reset project to clean slate
pnpm run reset-project

# Type generation from Supabase
supabase gen types typescript --project-id <project-id> > supabase/types/database.ts

# EAS Build
eas build --platform ios --profile preview
eas build --platform android --profile preview
```

## Project Structure

```
apps/mobile/ (Expo)
├── app/                    # Routes via expo-router (auth, onboarding, home, paywall)
├── lib/                    # Core integrations
│   ├── supabase.ts
│   ├── queryClient.ts
│   └── superwall.ts
├── features/               # Feature modules (onboarding, habits, chat, tracker)
├── components/ui/          # Paper component wrappers
├── stores/                 # Zustand stores (optional, minimal)
├── env.ts                  # Zod-validated config
└── types/                  # TypeScript types

supabase/
├── functions/              # Edge Functions
│   └── ai/index.ts        # Single AI endpoint with SSE streaming
├── migrations/            # SQL migrations
└── types/database.ts      # Generated DB types
```

## Architecture

### Routing Structure
- Uses Expo Router (file-based routing)
- Main app entry: `app/_layout.tsx` - Root layout with theme provider
- Tab navigation: `app/(tabs)/_layout.tsx` - Bottom tab configuration
- Auth flow: `app/(auth)/` - Sign in/up screens
- Paywall: `app/paywall.tsx` - Superwall integration
- 404 handling: `app/+not-found.tsx`

### Key Patterns
- **Path aliasing**: `@/` maps to project root (configured in tsconfig.json)
- **Theme handling**: React Native Paper theming (MD3)
- **Type safety**: Strict TypeScript + Zod schemas for runtime validation
- **API contract**: Zod schemas define DB/API contract
- **Platform-specific code**: `.ios.tsx` and `.web.ts` file extensions
- **Env validation**: All config through Zod-validated `env.ts`
- **Feature flags**: `remote_settings` table cached locally

### State Management & Data Flow
- **Server state**: TanStack Query with optimistic updates
- **Forms**: React Hook Form + Zod validation
- **Local storage**: MMKV for persistent data
- **Minimal global state**: Zustand only for small UI state
- **Supabase**: Typed client with generated types
- **Paywall**: Superwall entitlement checks gate premium features

## Database Schema (Core Tables)

```sql
-- Profiles table (extends auth.users)
create table profiles (
  id uuid primary key references auth.users on delete cascade,
  email text, name text, settings jsonb default '{}'::jsonb,
  created_at timestamptz default now()
);

-- Questionnaires for onboarding
create table questionnaires (
  id uuid primary key default gen_random_uuid(),
  slug text unique, schema_json jsonb not null
);

-- User answers to questionnaires
create table answers (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  questionnaire_id uuid references questionnaires(id) on delete cascade,
  payload_json jsonb not null, created_at timestamptz default now()
);

-- Habits tracking
create table habits (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  title text not null, schedule_json jsonb default '{}'::jsonb,
  created_at timestamptz default now()
);

-- Habit completion logs
create table habit_logs (
  id uuid primary key default gen_random_uuid(),
  habit_id uuid references habits(id) on delete cascade,
  date date not null, status text check (status in ('done','missed','skipped'))
);

-- Chat messages
create table messages (
  id bigserial primary key,
  user_id uuid references profiles(id) on delete cascade,
  role text check (role in ('system','user','assistant')),
  content text not null, created_at timestamptz default now()
);

-- Enable RLS on all tables
alter table profiles enable row level security;
alter table answers enable row level security;
alter table habits enable row level security;
alter table habit_logs enable row level security;
alter table messages enable row level security;

-- RLS policies (own rows only)
create policy "own rows" on answers
for all using (user_id = auth.uid()) with check (user_id = auth.uid());
-- Repeat for habits, habit_logs, messages
```

## AI Edge Function Pattern

```typescript
// supabase/functions/ai/index.ts
import "jsr:@supabase/functions-js/edge-runtime.d.ts";
import { streamText } from "jsr:@ai-sdk/openai";

Deno.serve(async (req) => {
  const { messages, model = "gpt-4o-mini", system } = await req.json();
  const apiKey = Deno.env.get("OPENAI_API_KEY")!;
  
  const stream = await streamText({
    model: { provider: "openai", name: model, apiKey },
    messages,
    system
  });
  
  return new Response(stream.toReadableStream(), {
    headers: { "Content-Type": "text/event-stream" }
  });
});
```

## 3-Day Development Plan

### Day 1: Foundation
- Scaffold app + Supabase setup + Edge Function
- Auth flow (sign up/in/out) with Supabase Auth
- Type generation from Supabase schema
- Basic navigation structure with expo-router
- Sentry + PostHog integration

### Day 2: Core Features
- Main screen implementation (habits/chat/tracker)
- Data models and Supabase integration
- React Query setup with optimistic updates
- Onboarding questionnaire (React Hook Form + Zod)
- AI integration via Edge Function

### Day 3: Polish & Ship
- Superwall paywall integration
- Push notifications setup
- Empty states, error handling, settings
- Paper theming and consistent UI
- EAS Build configuration
- TestFlight/Internal test deployment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peter-mach)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peter-mach)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
