## app-spree

> This repository is a pnpm workspace containing multiple Expo RN apps

Monorepo Guidance
This repository is a pnpm workspace containing multiple Expo RN apps
under /apps/<app-name>.
When chatting, scope all planning or code generation to the folder
currently selected in the Explorer (e.g. /apps/calmnest-headspace).
Never edit sibling apps unless explicitly instructed.

Project Tech Stack
Expo SDK 55 · React Native 0.74 · Expo Router v3 · TypeScript (strict) ·
Supabase backend (Auth, Postgres, Storage, Realtime, Edge Functions).
Styling: React Native StyleSheet or react-native-css.
Do NOT add Tailwind/NativeWind, Zod, Zustand, TanStack Query or any
other libraries unless I explicitly request them.

Folder Conventions
/apps/<name>/app → Expo Router routes (screens)

/apps/<name>/src → components, contexts, hooks, utils

/apps/<name>/supabase → SQL, edge functions, seed scripts
Business logic lives in src, not inside app screens.

Output Rules
Write/modify code directly; summarise large edits in plain English.

Do not introduce libraries beyond the approved stack.

Keep individual functions ≲ 80 LOC where practical.

Planning
When asked to PLAN, produce:
• vision · feature list · domain models · architecture diagram · folder
structure · two-sprint roadmap · ≥ 5 risks/mitigations · first 3 tasks.
Do NOT generate any code during planning.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aiden-Hyun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
