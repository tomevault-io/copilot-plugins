## pickkaru-vibe-exmaple

> - **Feature-First Clean Architecture**: Modularized directories by feature: `auth`, `track_setup`, `attendance`, `tracking`.

# Pickkaru AI Coding Rules & Directives

## Architecture & Code Structure
- **Feature-First Clean Architecture**: Modularized directories by feature: `auth`, `track_setup`, `attendance`, `tracking`.
- **UI & Logic Separation**: Keep presentation widgets stateless. State and events must live inside Riverpod StateNotifiers.
- **Repository Encapsulation**: Wrap all Firestore database operations in repository classes. Never invoke Firestore calls directly in UI files.
- **Strong Typing**: Write strict, strongly-typed Dart code. No `dynamic` type declarations. Always specify return types and parameter types.

## Crucial App Guardrails
- **Opt-Out Attendance Model**: Students are default set to attending ("Yes" status). They are only absent if their schedule entry or today's flag is explicitly set to false ("No" status).
- **Attendance Locking**: programmatically lock attendance toggles if `isTripActive` is true in the track state. Reject client mutations immediately.
- **State Isolation**: Never update state variables directly inside UI elements. All mutations go through state controllers/providers.

## Firestore Transactions & Security
- **Attendance Lock Check**: Reads the track document. If `isTripActive` is true, reject the mutation.
- **Track Assignment Check**: Verify `/tracks/{trackId}` exists before assigning it to a student user profile. If it does not exist, return an explicit error string to trigger a toast message.

## Mapbox Optimization
- Ensure Mapbox controllers only trigger viewport transitions or point updates upon explicit coordinate delta changes to optimize performance.

---
> Source: [jawadrao149/pickkaru-vibe-exmaple](https://github.com/jawadrao149/pickkaru-vibe-exmaple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
