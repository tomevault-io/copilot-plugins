## rir-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npx expo start          # Start dev server (scan QR with Expo Go)
npx expo start --android
npx expo start --ios
npx expo start --web

npm run lint            # ESLint on all .ts/.tsx
npm run format          # Prettier on src/ and app/
```

## Architecture

### Routing — Expo Router v4 (file-based)
All screens live in `app/`. The file path is the route. Tab navigation is in `app/(tabs)/`. Adding a screen = creating a file. The root layout (`app/_layout.tsx`) wraps the entire app with `QueryClientProvider`.

### Data fetching pattern (3 layers)
1. **`src/services/`** — pure async functions using the Axios instance (`src/services/api.ts`). No React, no hooks.
2. **`src/hooks/`** — TanStack Query hooks that wrap service calls (e.g. `usePosts`). This is where `queryKey`, `staleTime`, and `enabled` logic lives.
3. **Screens** — consume hooks directly, never call services directly.

### State management
- **Server state** (API data): TanStack Query only — do not store API responses in Zustand.
- **Client/UI state** (auth, preferences, UI flags): Zustand stores in `src/store/`. Stores are plain `create()` calls with no providers needed.
- `src/lib/queryClient.ts` holds the global `QueryClient` config (staleTime, gcTime, retry).

### Styling — NativeWind v4
Use `className` props with Tailwind utility classes. Custom design tokens (colors, etc.) are extended in `tailwind.config.js`. The `global.css` file is the Tailwind entry point loaded in `app/_layout.tsx`.

### Path alias
`@/*` resolves to `src/*`. Always use this alias for imports across the codebase (e.g. `import { Button } from '@/components/ui/Button'`).

### Adding a new API resource
1. Add types to `src/types/api.ts`
2. Create `src/services/<resource>.ts` with service functions
3. Create `src/hooks/use<Resource>.ts` with TanStack Query hooks
4. Use the hook in the screen — never import services directly in screens

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frangabriel13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
