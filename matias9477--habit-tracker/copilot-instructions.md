## habit-tracker

> - Use **SQLite** as the single source of truth for persistent data.


# Project Rules – Habit Tracker App

## 💡 Architecture

- Use **SQLite** as the single source of truth for persistent data.
- Use **Zustand** to store app-wide state (e.g. loaded habits, today’s completion status).
- Load data from SQLite on app start and hydrate Zustand.
- Save updates both to Zustand and SQLite to keep state and persistence in sync.

## ⚙️ State Management

- Create a `habitStore` in `/store` using Zustand.
- Avoid overusing Zustand — keep it focused on data loaded from the DB.
- UI state (like modals, navigation, toggles) can be local or use lightweight state hooks.

## 🧠 Data Layer

- Create one file per entity in `/db` (e.g. `habits.ts`, `completions.ts`) to manage:

  - schema creation
  - insert, update, delete helpers
  - queries for streaks or metrics

- Use TypeScript types to model rows and query results.

## 📱 UI Components

- Put reusable visual components (like `HabitCard`, `DateHeader`, `CheckButton`) in `/components`.
- Keep screens focused on layout and data orchestration (fetch → display → interact).

## 🧪 Code Quality

- Use TypeScript for all files, with clear types for props and DB models.
- Keep async/await logic clean and wrapped in helper functions.
- Use `try/catch` for all DB operations, and report errors to the console for now.
- Comment any logic that’s not obvious (e.g. streak calculation).

## 📅 Notifications

- Use `expo-notifications` and schedule reminders on habit creation (optional for MVP).
- Keep a helper module for notification logic (`utils/notifications.ts`).

## 🧹 Structure

- Avoid putting logic in `App.tsx` — use it only for:

  - initializing DB
  - setting up Zustand
  - wrapping Navigation and context providers

- Prefer flat folder structure unless complexity requires nesting.

---

Following this structure will make the app easier to expand later (e.g. cloud sync, auth, gamification).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matias9477) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
