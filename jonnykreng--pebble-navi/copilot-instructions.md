## pebble-navi

> Pebble smartwatch navigation app. C (watch) + TypeScript (phone JS).

# pebble-navi

Pebble smartwatch navigation app. C (watch) + TypeScript (phone JS).

## Build & run

```sh
npm run tsc          # TS compile only
npm run build        # tsc + pebble build (full)
npm run start        # build + install to emery emulator
npm run debug        # build + install + logs
npm run push         # build + install to phone
npm run format       # prettier --write src/**/*.ts
```

Prerequisites: Pebble SDK at `~/Library/Application Support/Pebble SDK/SDKs/current/`.

## Architecture

- **`src/c/`** — Watch C code (`main.c` entry, `navigation.c` map layer/bitmap rendering, `menu.c` menu overlay).
- **`src/pkjs/`** — Phone-side TypeScript, compiled to JS in the same directory (`.gitignore` ignores `src/**/*.js`).
- `server/` — OSM tile fetch, routing (OSRM), render to Pebble palette, localStorage cache.
- 
## Key quirks

- `tsconfig.json` uses `ignoreDeprecations: "6.0"` (TS 6.x + ES5 target).
- Message keys are defined in `package.json` `pebble.messageKeys`. `CMakeLists.txt` generates `message_keys.auto.h` from them (CLion IDE support only; real build uses `waf`).
- Targets: `emery`, `gabbro`.
- RLE-compressed bitmap chunks sent via `AppMessage`.
- No tests.

---
> Source: [JonnyKreng/pebble-navi](https://github.com/JonnyKreng/pebble-navi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
