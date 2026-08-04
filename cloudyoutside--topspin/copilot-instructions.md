## topspin

> TopSpin is an 8-player physics arena battler. The current priority is to get movement, spin, and collision feel right before matchmaking and UI are expanded.

# TopSpin — Roblox arena battler

TopSpin is an 8-player physics arena battler. The current priority is to get movement, spin, and collision feel right before matchmaking and UI are expanded.

## Project layout

**Script Sync (edit in Cursor, auto-syncs to Studio):**
- `ServerScriptService/` — server scripts such as `GameBootstrap.server.luau` and `Services/`
- `ReplicatedStorage/` — shared modules and config under `Config/`, `Shared/`, and `Remotes/`
- `StarterPlayerScripts/` — client entry points such as `BladeController.client.luau`

**MCP-only (not on disk — create/inspect via Roblox Studio MCP):**
- `StarterGui/` — HUD and spectator UI later in the build
- `Workspace/` — lobby, arena geometry, spawn points, kill boundaries
- `StarterPack/` — not used in the initial build; TopSpin uses a match transformation rather than a Tool

**MCP server name:** `user-Roblox_Studio`

**Not in Git:** `*.rbxlx` place files (local Studio save only)

## File naming

| Pattern | Studio type | Runs on |
|---|---|---|
| `Name.server.luau` | Script | Server |
| `Name.client.luau` | LocalScript | Client |
| `Name.luau` | ModuleScript | Required by server or client |

Wrong extension = script runs in the wrong context or not at all.

## How to work

- Read `ARCHITECTURE.md` before adding or changing systems. It is the source of truth for the current implementation.
- Update `ARCHITECTURE.md` in the same turn whenever you add, move, rename, or delete a script or MCP-managed instance.
- Build one phase at a time and verify it in Studio before moving onward.
- Use MCP to playtest and inspect console output instead of guessing at runtime issues.
- Keep server-authoritative gameplay logic in server scripts and use RemoteEvents only for player intent and state replication.

## Code rules

- Use `--!strict` and type annotations on functions.
- Client ↔ server: RemoteEvents / RemoteFunctions only. No `_G` globals.
- Minimal comments — only where the logic is non-obvious.
- No unused variables or parameters.
- Prefer modules in `ReplicatedStorage/` for shared logic; keep `.server.luau` and `.client.luau` entry points thin.

## Game design (target)

- Eight players join a match and fight in a physics-based arena.
- Players move with momentum-based WASD input and a visible spin meter.
- Collision causes knockback and spin loss, and players are eliminated when they spin out or ring out.
- The first arena is a circular Classic arena with a simple lobby loop.

## Docs map

| File | Purpose |
|---|---|
| `CLAUDE.md` | Project rules and workflow guidance |
| `ARCHITECTURE.md` | Living map of scripts and Studio instances |
| `TOPSPIN.md` | Phase-by-phase build guide |
| `DESIGN.md` | High-level design reference |
| `SETUP.md` | One-time Script Sync + MCP setup |

---
> Source: [CloudyOutside/TopSpin](https://github.com/CloudyOutside/TopSpin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
