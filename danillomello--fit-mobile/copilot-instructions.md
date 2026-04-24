## fit-mobile

> > NX | React Native 0.81 | Expo 54 | TypeScript

# ConnectHealth - fit-mobile

> NX | React Native 0.81 | Expo 54 | TypeScript

## Context

This is the **mobile app** for ConnectHealth platform.
Shared documentation lives in the **fit-common** GitHub repo: `DanilloMello/fit-common`.

## API Contract Rule — MANDATORY

Before **adding, modifying, or removing any API client call**, fetch the source of truth:

```
Tool: mcp__github__get_file_contents
  owner: DanilloMello
  repo: fit-common
  path: docs/API_REGISTRY.md
```

- **New API call** → the endpoint must exist in Section 2 (Implemented). Never call a Planned endpoint.
- **Changing a call** (path, payload, response shape) → verify the change is reflected in the registry first.
- **Dead code** → if a method in `*.api.ts` has no matching entry in Section 2, remove it.

The registry is the contract. Mobile adapts to it — not the other way around.

## Git Workflow — MANDATORY

Run **`/gitflow`** before any task that involves branching, committing, or opening a PR. It fetches the live conventions from fit-common and enforces them.

## How to start any task

1. **Run `/fit-mobile-overview`** — the master index skill that orchestrates everything:
   - Fetches coding standards and doc registry from fit-common
   - Points to the right spec files (architecture, patterns, ui-components, rules) based on your task

2. **Commands**:
   ```bash
   cd apps/mobile && npx expo start      # Start Metro — scan QR with Expo Go
   npx nx test                           # Run all tests
   eas build --platform android --profile preview     # APK for QA
   eas build --platform android --profile production  # AAB for Play Store
   ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DanilloMello) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
