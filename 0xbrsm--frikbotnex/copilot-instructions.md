## frikbotnex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FrikBotNex is a QuakeC bot AI system for Quake 1 deathmatch, evolved from FrikBot X (v0.10.2). It runs on the CleanFixedQuakeC base (Jason2Brownlee's cleaned GPL release of the original id Software game code).

## Build

QuakeC has no package manager or dependency system. Compile with any QuakeC compiler from the `src/qc/` directory:

```sh
cd src/qc
fteqcc    # or: gmqcc progs.src  /  qcc
```

The compilation manifest is `src/qc/progs.src`. Output goes directly to the repo root: `progs.dat` (game bytecode) and `progs.lno` (debug line numbers).

There are no automated tests — testing requires running in a Quake engine.

## Architecture

### Three source areas

1. **Base game code** (`src/qc/`) — Standard Quake game logic (monsters, weapons, items, physics, client). `defs.qc` must be compiled first (global declarations).

2. **Bot AI system** (`src/frikbot/`) — The core of this project:
   - `bot.qc` — Bot entity lifecycle, main loop, initialization
   - `bot_ai.qc` — Target selection, threat assessment, goal management, aim pipeline
   - `bot_think.qc` — Opponent modeling, team coordination, utility scoring
   - `bot_shoot.qc` — Weapon scoring, switching, fire control
   - `bot_fight.qc` — Combat movement state machine (fight/retreat/pressure/flank)
   - `bot_move.qc` — Movement command generation, bunny hopping
   - `bot_phys.qc` — Physics prediction and movement validation
   - `bot_way.qc` — Waypoint navigation and pathfinding
   - `bot_sense.qc` — Unified spatial awareness pass (compute once, read everywhere)
   - `bot_misc.qc` — Utility functions
   - `bot_ed.qc` — In-game console/editor interface
   - `bot_qw.qc` — QuakeWorld protocol variant (not compiled by default)

3. **Waypoint data** (`src/waypoints/`) — Per-map navigation graphs (`map_dm1.qc` through `map_dm6.qc`) with AI behavior flags.

### Bot integration into game loop

The bot system hooks into the base game at four points in `src/qc/world.qc` and `src/qc/client.qc`:
- `BotInit()` in `WorldSpawn()` — spawns physics entities for each client slot
- `BotFrame()` in `StartFrame()` — per-frame AI processing (pathfinding, physics, decisions)
- `BotPreFrame()` in `PlayerPreThink()` — early bot checks, skips human-player logic for bots
- `BotPostFrame()` in `PlayerPostThink()` — late-stage updates after engine processing

### Key entity fields (bot-specific)

- `b_skill` (0–3) — Bot difficulty level, affects aim, dodge timing, decision-making
- `b_aiflags` — Bitfield of `AI_*` constants controlling waypoint behavior (telelinks, doors, jumps, platforms, sniper spots, ambush points — 18 flags total)
- `target1`–`target4` — Tracked entities (enemies, items, goals)
- `phys_obj` — Linked physics prediction entity (separate from game physics)

### Compilation order matters

`progs.src` defines strict compile order. Waypoints and bot modules come before base game files (except `defs.qc`). New files must be added to `progs.src` in the correct position.

## Licensing

Base game code is GPL v2 (id Software). FrikBot code is public domain (Ryan "FrikaC" Smith).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xBrsm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/0xBrsm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
