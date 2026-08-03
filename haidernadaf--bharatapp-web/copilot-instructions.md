## bharatapp-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run dev      # Start development server at http://localhost:3000
npm run build    # Production build
npm run start    # Start production server
npm run lint     # Run ESLint
```

## Architecture Overview

This is a **Next.js 16 App Router** application for BharatApp, a professional networking platform with two modes: Personal (social networking) and Business (recruitment).

### Tech Stack
- Next.js 16.1.1 with App Router
- React 19, TypeScript 5
- Redux Toolkit + RTK Query for state management and API calls
- Tailwind CSS v4

### Directory Structure

```
app/
├── (auth)/         # Auth routes (login, register) - route group hidden from URL
├── (personal)/     # Personal mode routes (feed, profile, network, jobs, etc.)
└── business/       # Business mode routes (cabinet, team, settings)

components/
├── personal/       # PersonalSidebar, FeedPost, CreatePostModal
└── business/       # BusinessSidebar, modals for meetings/interviews

hooks/
├── useModeDetection.ts   # Detects Personal vs Business mode from URL
└── useSetupStatus.ts     # Tracks profile setup completion

lib/redux/
├── store.ts        # Redux store with RTK Query middleware
├── hooks.ts        # Typed useDispatch, useSelector
├── types.ts        # All TypeScript interfaces
├── slices/authSlice.ts   # Auth state + localStorage persistence
└── api/            # RTK Query API slices (authApi, postApi, connectionApi, generalApi)
```

### Key Patterns

**Route Groups**: `(auth)` and `(personal)` groups don't appear in URL paths.

**Dual-Mode UI**: Personal mode (`/`) and Business mode (`/business`) have separate layouts with sidebars. Mode is determined by URL, switchable via sidebar toggle.

**State Management**: Redux Toolkit with RTK Query. All imports available from `@/lib/redux`:
```tsx
import { useAppSelector, useAppDispatch, useLoginUserMutation } from "@/lib/redux";
```

**Auth Flow**: Registration → Profile Setup (`/profile-setup`) → Home. Token and user auto-persist to localStorage and restore on app load.

**API Layer**: RTK Query with base URL `https://bapp.marketsverse.com`. Bearer token auto-injected via `prepareHeaders`. Tag-based cache invalidation.

### Path Alias
`@/*` maps to project root (e.g., `@/lib/redux`, `@/components/personal/`)

### Styling
Tailwind CSS utility classes. Most pages are client components (`"use client"`).

**Color Scheme by Mode:**
- **Personal Mode**:
  - Primary: `#FB923C` (orange), Hover: `#F97316`, Dark: `#C2410C`, Tints: `#FFF7ED` / `#FFEDD5` / `#FED7AA`
  - Text: `#1E293B` (slate-800), Secondary: `#64748B` (slate-500)
  - Borders: `#E2E8F0` (slate-200), Background: `#F8FAFC` (slate-50)
- **Business Mode**: `#2A9D8F` (teal/green)
- **Auth Pages**: `#f56e3d` (brand orange) with warm tones

---
> Source: [HaiderNadaf/Bharatapp-web](https://github.com/HaiderNadaf/Bharatapp-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
