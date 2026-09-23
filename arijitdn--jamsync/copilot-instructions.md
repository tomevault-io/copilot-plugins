## jamsync

> JamSync contains an Expo SDK 57 client and LAN relay:

# Repository Guidelines

## Project Structure & Module Organization

JamSync contains an Expo SDK 57 client and LAN relay:

- `App.tsx` contains the UI, Socket.IO client, clock calibration, and synchronization logic.
- `index.ts` is the Expo entry point.
- `server/index.js` implements the Express/Socket.IO room and timeline relay.
- `assets/` stores application icons and splash artwork.
- `app.json` configures Expo, native identifiers, and background audio.
- `eas.json` defines preview APK and production build profiles.

Do not commit generated `node_modules/`, `.expo/`, `dist/`, `android/`, or `ios/` directories.

## Architecture Overview

The relay owns rooms and a timestamped playback timeline. Clients estimate relay-clock offset, schedule commands, and correct drift locally. Keep protocol changes compatible between `App.tsx` and `server/index.js`. Enforce host-only actions on the server.

## Build, Test, and Development Commands

- `npm install` installs locked dependencies.
- `npm run dev` starts the relay and Expo concurrently.
- `npm run relay` starts only the LAN relay on port `3030`.
- `npm start` opens the Expo development server.
- `npm run android` / `npm run ios` launches a platform target.
- `npm run typecheck` runs strict TypeScript validation.
- `npx expo export --platform android` verifies production bundling.
- `npx eas-cli build --platform android --profile preview` creates an installable APK.

## Coding Style & Naming Conventions

Use TypeScript for client code, two-space indentation, single quotes, semicolons, and trailing commas in multiline structures. Use `PascalCase` for components and types, `camelCase` for functions and values, and descriptive events such as `room:create`. Keep synchronization constants centralized. No formatter or linter is configured; follow surrounding style.

## Testing Guidelines

There is no automated test framework yet. Every change must pass type checking and an Expo export. For networking or timing changes, test one host and one guest on the same Wi-Fi, covering create, join, playback, seek, disconnect, and host departure. Future tests should use `*.test.ts` or `*.test.tsx`.

## Commit & Pull Request Guidelines

History currently contains only `Initial commit`, so use concise imperative commits, for example `Fix drift correction timer`. Pull requests should explain behavior changes, list validation performed, link relevant issues, and include screenshots for UI changes. Call out protocol or build-configuration changes explicitly.

## Configuration & Agent Notes

Do not commit credentials, signing files, or local `.env` files. Before changing Expo APIs, consult the exact [Expo SDK 57 documentation](https://docs.expo.dev/versions/v57.0.0/); do not assume APIs from newer SDKs apply.

---
> Source: [arijitdn/jamsync](https://github.com/arijitdn/jamsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
