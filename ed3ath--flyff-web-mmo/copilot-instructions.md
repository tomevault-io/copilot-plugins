## flyff-web-mmo

> Project-level guidance for AI assistants working in this repository.

# CLAUDE.md

Project-level guidance for AI assistants working in this repository.

## Commands

```bash
pnpm install                    # install all workspace deps
pnpm build                      # tsc all packages
pnpm test                       # node --test across all packages
pnpm dev:login                  # login server :28000 (tsx watch)
pnpm dev:game                   # game server :2000 (tsx watch)

# Database
docker-compose up -d postgres   # PostgreSQL 16 (or use SQLite: DATABASE_URL=sqlite:./data/flyff.db)
npx tsx scripts/create-account.ts <user> <pass> <email>

# Single test
cd packages/shared && node --test --import tsx src/protocol/binary.test.ts
cd packages/game-server && node --test --import tsx src/skill/SkillDefinition.test.ts

# WASM client (separate toolchain — not part of pnpm build)
pnpm client:install               # one-time emsdk setup
pnpm client:configure             # emcmake cmake (Ninja)
pnpm client:build                 # compile to flyff_web.wasm
pnpm client:run                   # serve at :8080
pnpm lnd:convert                # convert .lnd map data → terrain assets
```

## Gotchas

- **No linter.** `pnpm lint` silently no-ops — no package defines a lint script.
- **No CI.** No GitHub Actions or other CI config exists.
- **ESM only.** All packages `"type": "module"`. Imports use `.js` extensions for `.ts` files (moduleResolution: bundler).
- **Tests use Node built-in runner** (`node --test`), not jest/vitest.
- **DB migrations auto-apply** on first Docker container creation only. `docker-compose down -v` to re-run.
- **SQLite fallback** for local dev — set `DATABASE_URL=sqlite:./data/flyff.db` in `.env`, no Docker needed.
- **`reference/`** is original Flyff v15 C++ source (read-only reference for game mechanics).

## Architecture

pnpm workspace monorepo. Dependency direction: `shared` → `server-core` → `login-server` / `game-server`. The C++/WASM client (`packages/client/`) consumes `@flyff/shared` types via generated headers and does not depend on `server-core`.

| Package | Role |
|---------|------|
| `@flyff/shared` | Binary protocol (CAr format), types, constants — zero deps |
| `@flyff/server-core` | DB (PG + SQLite), data loaders, crypto, net — pg, better-sqlite3, ws |
| `@flyff/login-server` | Auth, character management — WS :28000 |
| `@flyff/game-server` | World simulation, combat, AI, snapshots — WS :2000 |
| `tools/` | Asset conversion (DDS, O3D, RES, animations, terrain) |
| `packages/client/` | C++ Neuz client → WebAssembly (126 source files, cmake+ninja+emsdk) |

### Key design decisions

- **Binary protocol** matches original CAr serialization: little-endian, `[DWORD opcode][payload]`, strings as `int32` length + UTF-8.
- **Server-authoritative.** All game logic on server; client renders only.
- **Snapshot batching**: `[DWORD SNAPSHOT][DWORD playerId][WORD count]{entries}`.
- **Entity IDs** are sequential integers.
- **Fixed-tick server loop** for world simulation.
- **Spatial hash grid** for entity visibility.

### WASM client architecture

D3D9→GLES2.0 shim translates DirectX 9 to WebGL. MFC stripped to STL stubs. WebSocket replaces DirectPlay. In-canvas HUD rendered via GLES2 quad batching (not DOM). Text via stb_truetype baked atlas.

**Pre-include chain** (neuz_stdafx.h): mfc_strip.h → neuz_stubs.h → d3d9_gles.h → Obj.h/Ctrl.h → ProjectCmn.h → Ar.h → Item.h → pocket.h → mover.h → scanner.h/Script.h → flyffevent.h → resData.h → WndBase.h/WndManager.h/WndControl.h/WndNeuz.h → neuz_game_stubs.h

**Stub pattern**: stub classes guarded with `#ifndef __GUARD__` / `#define __GUARD__` to prevent redefinition. `#pragma once` headers kept as local stubs. CProject/CLandscape claim real headers' include guards to skip them.

**HUD renderer**: the **MFC bridge is canonical** (`mfc_render_bridge.cpp` creates CWndBase windows, routes mouse/wheel events, renders via `m_wndOrder`). `canvas_hud.cpp` is deprecated — do not extend it.

**Terrain**: double-sided with `gl_FrontFacing` back-face darkening (`if (!gl_FrontFacing) lighting *= 0.25;` in the fragment shader, terrain pass only). NEVER darken via `normal.y < 0` — that hits character meshes too. NEVER procedurally generate ground textures — they're data-driven from `.lnd` (see `tools/src/lnd-to-terrain/`). Heightmap flags: `HGT_NOWALK=1000, NOFLY=2000, NOMOVE=3000, DIE=4000` — subtract the matching flag base.

**Player spawn**: original Flaris revival coords (`pos_x=6968, pos_z=3329, pos_y=100`, tile 13,6) per `WdMadrigal.rgn` line 745. Client snaps Y to terrain every frame.

**Ninja data-repack gotcha**: after `pnpm lnd:convert` or asset-only edits, `pnpm client:build` skips the data bundle repack. Force a re-link with `touch packages/client/platform/main.cpp` before `pnpm client:build`.

## Code style

- TypeScript strict mode, ES2022 target, ESNext modules
- 2-space indent (TS/JS/JSON), tabs (C++/Makefile), LF line endings
- camelCase vars/functions, PascalCase classes/types/interfaces
- No eslint/prettier — `.editorconfig` only

## Verification

After changes:
1. `pnpm build` — type-checks and compiles all packages
2. `pnpm test` — runs protocol serialization + skill tests
3. For WASM changes: `pnpm client:build` then hard-refresh browser

---
> Source: [ed3ath/flyff-web-mmo](https://github.com/ed3ath/flyff-web-mmo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
