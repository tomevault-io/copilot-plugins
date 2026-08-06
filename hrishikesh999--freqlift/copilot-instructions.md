## freqlift

> Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing native/Expo code.

# Freqlift — agent notes

## Expo has changed
Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing native/Expo code.

## What this is
Freqlift — an emotional-habit tracker (React Native + Expo, TypeScript). Users get periodic "check-ins", score their day (binary positive/negative), and get "the Lift" (a nudge one step up) after a negative check-in. Local-first: no accounts, no cloud, works offline. Full spec is the PRD + design handoff in `~/Dropbox/FreqLift/design_handoff_freqlift_mobile_app/` (Soft Dawn is the approved design).

## Architecture (`src/`)
- `theme/tokens.ts` — all colors/type/spacing (Soft Dawn). Import these; don't hardcode hex.
- `content/` — emotions (12 categories + nuance + reach map), quotes (60 original), greetings, score bands.
- `db/` — expo-sqlite. `check_ins` is the source of truth and **insert-only** (immutability, PRD 7.7 — no update/delete UI ever). `daily_scores` is a disposable cache. Sync-ready: UUID ids, UTC `created_at`, nullable `synced_at`.
- `scheduling/` — pure slot math (`slots.ts`), day-close/reconcile, check-in target + 3-extra/day cap.
- `notifications/` — local scheduled notifications, tap routing, permission. Custom chime swap point: `CUSTOM_SOUND` in `notifications.ts`.
- `email/` — MailerLite capture (queue + retry). Config via `EXPO_PUBLIC_MAILERLITE_*` (see `.env.example`).
- `screens/`, `navigation/`, `state/`, `components/` — UI.

## Conventions
- Scores/reports are always recomputed from raw `check_ins`; never trust an aggregate as truth.
- A check-in row's `local_date` is frozen at write time (timezone-change safe).
- Verify changes with `npx tsc --noEmit` and `npx expo export --platform ios` (catches bundling errors; no full Xcode in this env).

## Deferred / owner action items
- Custom notification chime asset (using OS default; `CUSTOM_SOUND`).
- MailerLite API token + group id in `.env` (capture queues locally until set).
- P1 not built: dark mode, streak-freeze grace day, monthly history view.

---
> Source: [hrishikesh999/freqlift](https://github.com/hrishikesh999/freqlift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
