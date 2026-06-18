## meeting-wizard-codebase

> Described is an AI-powered meeting notes app. It has two parts:

# Described - AI Meeting Notes App

## Overview

Described is an AI-powered meeting notes app. It has two parts:
- **`meeting-wizard/`** - Next.js 14 web app (dashboard, meeting management, settings)
- **`meeting-wizard-ext/`** - Chrome extension for recording browser tab audio

## Tech Stack

- **Framework:** Next.js 14 (App Router), React 18, TypeScript 5
- **Backend/Auth:** Supabase (PostgreSQL + Auth with Google OAuth)
- **AI:** OpenAI GPT-4o-mini for transcript processing
- **Styling:** Tailwind CSS 3.4, shadcn/ui (base-nova style), CSS variables for theming
- **Fonts:** Geist (sans, local), Instrument Serif (serif, Google Fonts)
- **Icons:** Lucide React
- **Other:** date-fns, Zod, jsPDF, class-variance-authority

## Project Structure

```
meeting-wizard/
├── app/
│   ├── page.tsx                    # Landing page
│   ├── layout.tsx                  # Root layout (fonts, theme provider)
│   ├── globals.css                 # CSS variables, theme colors
│   ├── api/
│   │   ├── process/route.ts        # Transcript -> AI processing -> DB storage
│   │   └── calendar/sync/route.ts  # Fetch Google Calendar events
│   ├── auth/callback/route.ts      # OAuth callback, stores Google tokens
│   └── (app)/                      # Protected route group
│       ├── dashboard/page.tsx      # Stats, upcoming meetings, calendar events
│       ├── meetings/
│       │   ├── page.tsx            # Meeting list with filters
│       │   └── [id]/page.tsx       # Meeting detail (transcript, notes, export)
│       └── settings/page.tsx       # Profile, preferences, integrations
├── components/
│   ├── ui/                         # shadcn components (button, card, input, etc.)
│   ├── layout/                     # AppShell, Sidebar, Header, MobileNav, BottomDock
│   ├── dashboard/                  # StatsCards, UpcomingMeetings, RecentActivity
│   ├── meetings/                   # MeetingDetail, MeetingTable, QuickAdd
│   ├── settings/                   # ProfileSettings, PreferenceSettings, IntegrationSettings
│   └── shared/                     # ThemeProvider
├── lib/
│   ├── supabase/                   # client.ts, server.ts, middleware.ts
│   ├── google-calendar.ts          # Google Calendar API (fetch events, token refresh)
│   ├── queries.ts                  # Supabase DB queries
│   ├── constants.ts                # App constants
│   └── utils.ts                    # cn() helper
├── types/database.ts               # TypeScript types for all entities
├── supabase/migrations/            # SQL migration files
└── middleware.ts                   # Auth guard, session refresh

meeting-wizard-ext/
├── manifest.json                   # Manifest V3
├── popup.html/js                   # Extension UI (record controls, history, tutorial)
├── background.js                   # Service worker (manages offscreen doc, storage)
├── offscreen.html/js               # MediaRecorder for tab audio capture
└── icons/                          # Extension icons
```

## Database Schema (Supabase PostgreSQL)

**users** - extends auth.users
- `id`, `email`, `full_name`, `avatar_url`
- `auto_join_enabled`, `email_notifications`, `default_export_format`
- `google_access_token`, `google_refresh_token`, `google_token_expires_at`, `google_calendar_connected`

**meetings**
- `id`, `user_id` (FK), `title`, `scheduled_at`, `duration_seconds`
- `google_meet_link`, `status` (enum: scheduled/joining/recording/processing/ready/failed)
- `participants` (JSONB)

**meeting_notes**
- `id`, `meeting_id` (FK), `summary`, `transcript` (JSONB)
- `action_items` (JSONB), `key_points` (JSONB), `decisions` (JSONB)

Row Level Security: users can only access their own data.

## Key Flows

**Auth:** Google OAuth via Supabase -> `/auth/callback` -> stores Google tokens in users table -> redirect to dashboard

**Meeting Processing:** POST `/api/process` with transcript text -> GPT-4o-mini extracts title, summary, action items, key points, decisions -> stores in meetings + meeting_notes tables

**Calendar Sync:** GET `/api/calendar/sync` -> uses stored Google tokens -> fetches upcoming events with Meet links

**Chrome Extension Recording:** Popup captures tab audio via `chrome.tabCapture` -> streams to offscreen document -> MediaRecorder records audio/webm -> download as file

## Design System

- **Theme:** Light/dark mode (class-based toggle), warm neutral palette (beige/cream base)
- **Primary color:** #0D7FFF (blue)
- **Components:** shadcn/ui with base-nova style
- **Layout:** Sidebar navigation on desktop, bottom dock on mobile
- **All colors defined as CSS variables** in `globals.css` and mapped in `tailwind.config.ts`

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
OPENAI_API_KEY
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
```

## Commands

```bash
cd meeting-wizard && npm run dev    # Start dev server
cd meeting-wizard && npm run build  # Production build
```

---
> Source: [khushxxl/meeting-wizard-codebase](https://github.com/khushxxl/meeting-wizard-codebase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
