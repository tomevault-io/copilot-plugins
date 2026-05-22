## apppath

> React Native (Expo) mobile app with Cloudflare Worker backend. The app uses Expo Router for file-based navigation, Firebase for auth/database, and a Cloudflare Worker for the API layer.

# AppPath - AppPath

## Overview
React Native (Expo) mobile app with Cloudflare Worker backend. The app uses Expo Router for file-based navigation, Firebase for auth/database, and a Cloudflare Worker for the API layer.

## Commands
- `npm start` — Start Expo dev server
- `npm run web` — Start web version
- `cd worker && npm run dev` — Run Cloudflare Worker locally
- `cd worker && npm run deploy` — Deploy Worker to Cloudflare

## Architecture
- **Routing**: Expo Router (file-based) in `app/` directory
  - `app/(auth)/` — Login, signup, forgot password
  - `app/(tabs)/` — Main tab screens (home, notes, tasks, chat, profile)
  - `app/features/` — Additional feature screens (interns add here)
- **Shared code**: `src/` directory
  - `src/components/` — Reusable UI (Button, Input, Card, FeatureCard, EmptyState)
  - `src/contexts/` — AuthContext and ThemeContext providers
  - `src/constants/` — Theme colors/spacing, feature definitions
  - `src/types/` — TypeScript interfaces (User, Note, Task, ChatMessage)
  - `src/config/` — Firebase configuration
- **Backend**: `worker/` directory — Cloudflare Worker with REST API routes

## Code Style
- TypeScript strict mode
- Functional components with hooks
- Use `useTheme()` for all colors — never hardcode colors in screens
- Use constants from `src/constants/theme.ts` for spacing, font sizes, border radius
- Components accept style overrides via `style` prop

---
> Source: [SJA-Pathway/AppPath](https://github.com/SJA-Pathway/AppPath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
