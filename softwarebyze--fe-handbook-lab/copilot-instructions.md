## fe-handbook-lab

> FE Handbook Lab is a cross-platform (iOS, Android, Web) Expo SDK 54 + React Native educational app for the FE exam. All content is statically bundled — no backend, no database, no external APIs.

# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

FE Handbook Lab is a cross-platform (iOS, Android, Web) Expo SDK 54 + React Native educational app for the FE exam. All content is statically bundled — no backend, no database, no external APIs.

### Dev commands

| Task | Command |
|---|---|
| Install deps | `npm install` |
| Type check | `npx tsc --noEmit` |
| Dev server (web) | `npx expo start --web --port 8081` |
| Static export | `npx expo export --platform web` |

There is no ESLint config or separate lint script in the project; TypeScript strict mode (`npx tsc --noEmit`) is the primary static check.

### Running on Linux / Cloud VMs

- Use `--web` mode since there are no iOS/Android emulators available.
- Metro bundler takes ~15 s on first load; the app is ready when `Bundled ... entry.js` appears in the terminal.
- Port 8081 is the Expo default; pass `--port <N>` to change.

### Key paths

- `app/` — file-system based routes (expo-router v6)
- `data/corpus.ts` — static topic/formula/quiz data
- `data/lessonBlocks.ts` — lesson content blocks
- `lib/srs.ts` — SM-2 spaced-repetition logic
- `components/` — shared UI components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/softwarebyze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
