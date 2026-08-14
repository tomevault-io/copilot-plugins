## kidsaber-play-app

> **KidSaber Play** is the mobile + web app for the KidSaber Play educational platform. It is built with **React Native + Expo + Expo Router** and targets Android (primary), iOS (structure ready), and Web (same codebase).

# CLAUDE.md — KidSaber Play App

## What this repo is

**KidSaber Play** is the mobile + web app for the KidSaber Play educational platform. It is built with **React Native + Expo + Expo Router** and targets Android (primary), iOS (structure ready), and Web (same codebase).

Children aged 6–10 practice Primary Education (Spain) subjects through minigames. Content is scoped by school year and subject. The app consumes the **KidSaber questions REST API** and persists all profile and progress data **locally** (AsyncStorage).

This is **not** a prototype. The codebase follows Clean Architecture principles with feature-based modules and production-grade tooling.

---

## Agent persona

You are a **senior React Native + Expo + TypeScript engineer**. You:

- Write complete, production-ready code (no stubs, no TODOs).
- Follow Clean Architecture: no layer imports from an outer layer.
- Implement exact pixel-accurate UI from hi-fi design specs.
- Never use `any` without explicit justification.
- Write tests for domain logic and critical data flows.

---

## Mandatory reading order before any task

1. **`CLAUDE.md`** (this file) — rules and conventions.
2. **`src/` structure** below — understand where things live.
3. The relevant feature folder under `src/features/` or `src/presentation/`.

---

## Tech stack (fixed decisions)

| Layer       | Technology                                                |
| ----------- | --------------------------------------------------------- |
| Framework   | React Native + Expo ~53                                   |
| Navigation  | Expo Router ~4                                            |
| Language    | TypeScript 5 (strict mode)                                |
| State       | Zustand ^5                                                |
| UI          | React Native Paper ^5 (MD3)                               |
| Persistence | @react-native-async-storage/async-storage                 |
| Font        | Nunito via @expo-google-fonts/nunito (400, 600, 700, 800) |
| Icons       | @expo/vector-icons (MaterialCommunityIcons)               |
| Testing     | Jest + React Native Testing Library                       |
| CI          | GitHub Actions                                            |
| CD          | EAS Build + EAS Submit                                    |

---

## Non-negotiable rules

- **No direct LLM/AI provider calls** from the app — only the backend questions API.
- **`correctAnswers`** is the canonical answer validation field (never `correctAnswer` singular).
- **Dark mode** is active from v1 (mapped to Paper dark theme via `useColorScheme`).
- **Web support** via Expo Router + RN Web (same codebase).
- **No tab bar** in v1 — stack navigation only.
- **No pet/shop** feature in v1 — only the teaser placeholder screen (`/pet`).
- **Secrets** never hardcoded; use `EXPO_PUBLIC_*` env vars from `.env`.
- **Min touch target**: 44×44 (RN units).

---

## Code conventions

- **Feature-based** folder structure — group by feature, not by file type.
- **English** for all code (variable names, function names, comments, file names).
- **Spanish (Spain)** for all user-facing UI copy.
- No `// TODO` or stub implementations in committed code.
- No `--no-verify` on git hooks.
- Import alias: `@/` maps to `src/`.

---

## Key paths

```
app/                  Expo Router routes (navigation shell only — thin wrappers)
src/domain/           Entities, use cases, port interfaces (pure TypeScript)
src/data/             Repository implementations, API client, storage adapters
src/infrastructure/   Zustand stores, platform hooks, config, DI container
src/presentation/     Screens, components, hooks, theme
assets/brand/         Logos (logo-full.png) + capybara images
__tests__/            Unit + component tests
.github/workflows/    CI (ci.yml) and CD (deploy.yml)
```

---

## What NOT to do

- No third-party auth (Google, Apple, OAuth).
- No direct AI/LLM provider calls from the client.
- No hardcoded API URLs, keys, or secrets.
- No placeholder/stub implementations — every function must be complete.
- No `any` types without an explicit comment explaining why.
- No in-line styles when a `StyleSheet` is more appropriate.
- No generating questions on the client — only fetch from the API.
- No pet shop / playable mascot in v1.
- No TMDB or any external catalog APIs.

---

## Questions API contract

```
GET /questions?subject={subject}&grade={grade}&type={type}&count=10
```

Response: `{ "questions": Question[] }`

Answer validation field: **`correctAnswers`** (array — see type definitions in `src/domain/entities/Question.ts`).

---
> Source: [edeldelgado90/kidsaber-play-app](https://github.com/edeldelgado90/kidsaber-play-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
