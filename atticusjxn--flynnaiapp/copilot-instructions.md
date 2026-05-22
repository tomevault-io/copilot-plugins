## flynnaiapp

> FlynnAI now centres on the flynn concierge experience—an AI receptionist that answers calls, captures event details, and hands structured summaries back to teams. Screens and copy should reflect “events” (formerly “jobs”) and highlight receptionist controls such as voice selection, greeting scripts, and intake questions.

# Repository Guidelines

## Product Focus
FlynnAI now centres on the flynn concierge experience—an AI receptionist that answers calls, captures event details, and hands structured summaries back to teams. Screens and copy should reflect “events” (formerly “jobs”) and highlight receptionist controls such as voice selection, greeting scripts, and intake questions.

## Project Structure & Module Organization
The Expo app lives in `src/`, with feature screens under `src/screens` and shared UI widgets in `src/components`. Domain data models sit in `src/data` and `src/types`, while API and Supabase accessors are under `src/services`. Theme tokens live in `src/theme` and cross-cutting helpers in `src/utils`. Native shells for device builds are kept in `android/` and `ios/`. Assets such as icons and fonts are in `assets/`, and product briefs plus integration notes live in `docs/`. Twilio helper functions are maintained separately in `flynnai-lookup/`; keep its dependencies isolated from the mobile app.

## Build, Test, and Development Commands
Install dependencies with `npm install`. Use `npm start` (alias for `expo start`) for the Metro dev server, then pair with `npm run ios`, `npm run android`, or `npm run web` as needed. `expo start --clear` is helpful when Metro cache issues appear. When working inside `flynnai-lookup/`, run `npm install` and `npm run deploy` per its README before pushing lookup changes.

## Coding Style & Naming Conventions
The project is TypeScript-first. Use 2-space indentation, single quotes, and trailing commas where Prettier (shipped with Expo) formats automatically. Screens and components should be PascalCase (`ClientDetailsModal.tsx`), hooks camelCase, and util modules descriptive (e.g., `timeFormatter.ts`). Keep React state local to components unless shared context belongs in `src/context`. Update theme tokens rather than hard-coding colors.

## Testing Guidelines
Automated testing is not yet wired; when adding Jest or React Native Testing Library, place specs beside the code within `__tests__` folders and expose them through an `npm test` script. Until then, exercise new flows through Expo Go or a development build on device. Document manual QA steps in PRs, especially around Supabase writes and Twilio lookups.

## Commit & Pull Request Guidelines
Commits in history are single-purpose, sentence-case summaries (e.g., “Improve communication modals”). Follow that style, keep bodies optional but include context when touching Supabase schemas or Twilio logic. PRs should link to Shortcut tickets, outline implementation notes, call out environment or schema changes, and attach screenshots or screen recordings for UI updates. Request review from the mobile lead and note any manual test coverage performed.

---
> Source: [atticusjxn/FlynnAIapp](https://github.com/atticusjxn/FlynnAIapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
