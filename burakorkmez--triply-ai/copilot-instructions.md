## triply-ai

> Read the exact versioned docs at https://docs.expo.dev/versions/v56.0.0/ before writing any code.

# Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v56.0.0/ before writing any code.

# Stack

**App**

- Expo SDK 56 · React Native 0.85 · React 19.2 · TypeScript (strict)
- Expo Router (file-based routing)
- NativeWind v4 (styling)

**Backend & services**

- **Database:** Postgres on [Neon](https://neon.tech)
- **ORM:** [Drizzle](https://orm.drizzle.team)
- **Auth:** [Clerk](https://clerk.com)
- **Images:** [ImageKit](https://imagekit.io) for image optimization/delivery
- **Background jobs:** [Inngest](https://www.inngest.com)
- **Error tracking & monitoring:** [Sentry](https://sentry.io)

# Conventions

- **NEVER run the app yourself** (`npm run ios`/`android`/`start`, `expo start`, etc.). The dev server is always already running in a separate terminal. Do not start, restart, or build the app.
- **Native tabs are mandatory.** Always use native tabs for tab navigation — never a JavaScript/custom tab bar. This is a hard rule for this project.
- **Styling:** NativeWind v4 — use `className`, NOT `StyleSheet.create`.
- **Routing:** Expo Router, file-based. Screens live in `src/app`. Typed routes are enabled.
- **UI:** Native-first. Prefer `@expo/ui`, `expo-symbols` (SF Symbols), and `expo-glass-effect` over custom views.
- **Images:** use `expo-image` for rendering; serve through ImageKit for optimization.
- **Auth:** Clerk — use `@clerk/clerk-expo`. Store tokens via `expo-secure-store` token cache.
- **React Compiler is ON** — do NOT manually add `useMemo` / `useCallback` / `React.memo`.
- **Imports:** use the `@/` alias (`@/*` → `src/*`).

# Commands

- `npm run ios` / `npm run android` — native dev build (uses expo-dev-client)
- `npm run start` — dev server
- `npm run lint`
</content>

</invoke>

---
> Source: [burakorkmez/triply-ai](https://github.com/burakorkmez/triply-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
