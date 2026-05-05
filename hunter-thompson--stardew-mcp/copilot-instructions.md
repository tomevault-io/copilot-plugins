## stardew-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Stardew Valley MCP Bridge - A hybrid AI-controlled game mod that bridges Stardew Valley with AI assistants via the Model Context Protocol (MCP). Enables autonomous AI agents to control and play Stardew Valley through:
- A C# SMAPI mod running inside the game
- A Go MCP server that communicates with GitHub Copilot SDK (Claude Sonnet)
- WebSocket-based real-time game state synchronization

## Build Commands

### C# Mod (SMAPI)
```bash
cd mod/StardewMCP
dotnet build                    # Compile to StardewMCP.dll
```

### Go MCP Server
```bash
cd mcp-server
go build -o stardew-mcp         # Compile binary
./stardew-mcp                   # Run with default settings
./stardew-mcp -auto=false       # Connect without starting autonomous agent
./stardew-mcp -goal "Clear the farm" -url ws://localhost:8765/game
```

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│ STARDEW VALLEY (Game)                                   │
│   SMAPI Mod (C# .NET 6)                                 │
│     ModEntry → GameStateSerializer                      │
│              → CommandExecutor (w/ Pathfinder)          │
│              → WebSocketServer                          │
└─────────────────────────────────────────────────────────┘
              ↕ ws://localhost:8765/game
┌─────────────────────────────────────────────────────────┐
│ MCP Server (Go)                                         │
│   GameClient: WebSocket connection, state tracking      │
│   StardewAgent: 12 tools, autonomous loop, LLM calls    │
└─────────────────────────────────────────────────────────┘
              ↕ Copilot SDK
┌─────────────────────────────────────────────────────────┐
│ Claude Sonnet (via GitHub Copilot SDK)                  │
└─────────────────────────────────────────────────────────┘
```

### C# Mod Components (`mod/StardewMCP/`)

- **ModEntry.cs**: SMAPI entry point. Initializes WebSocket server, wires components, registers game loop events (Update, OneSecond, SaveLoaded). Broadcasts game state every 1 second, processes commands each frame.

- **CommandExecutor.cs**: Executes game commands (move_to, use_tool, select_item, etc.). Contains A* pathfinding integration, continuous movement processing, tool cooldown tracking (30 ticks between swings). Queue-based command processing - one command per game tick.

- **GameStateSerializer.cs**: Captures complete game state: Player, Time, World, Surroundings. Generates 61x61 ASCII map vision (30-tile scan radius). Serializes NPCs, items, terrain, quests, relationships, skills.

- **WebSocketServer.cs**: Server on `ws://localhost:8765/game`. Message types: "command", "get_state", "ping". Response types: "state", "response", "error", "pong".

- **Pathfinder.cs**: A* algorithm for navigation. 4-directional movement, 50,000 iteration limit, Manhattan distance heuristic. Checks walkability across tiles, objects, terrain features, buildings, furniture, water.

### Go MCP Server Components (`mcp-server/`)

- **main.go**: GameClient WebSocket manager with reconnection logic (5-second retry). GameState struct definitions for all game entities. Keep-alive pings every 15 seconds, 15-second command timeout.

- **copilot_agent.go**: StardewAgent using GitHub Copilot SDK. 12 standard tools (move_to, get_surroundings, interact, use_tool, use_tool_repeat, face_direction, select_item, switch_tool, eat_item, enter_door, find_best_target, clear_target) plus 30+ cheat mode tools. Contains embedded game knowledge (ASCII map legend, seed IDs, survival rules). Autonomous loop with emergency handling (time/energy checks), 60-second LLM call timeout.

## WebSocket Protocol

**Request**:
```json
{"id": "uuid", "type": "command", "action": "move_to", "params": {"x": 10, "y": 20}}
```

**Response**:
```json
{"id": "uuid", "type": "response", "success": true, "message": "...", "data": {...}}
```

## Key Design Patterns

- **Queue-based command execution**: One command per game frame prevents desync
- **Async state broadcasting**: Game state sent every 1 second, separate from command processing
- **Path recalculation**: Up to 5 attempts if initial pathfinding fails
- **Tool cooldown**: 30-tick gaps between tool swings (0.5s at 60fps)
- **Mutex-protected tool execution**: Prevents concurrent tool usage in agent
- **Embedded knowledge base**: Game mechanics, seed IDs, and survival rules baked into copilot_agent.go

## Cheat Mode

Cheat mode provides instant god-mode capabilities. Must call `cheat_mode_enable` before using any cheat commands.

**Categories:**
- **Mode Control**: cheat_mode_enable, cheat_mode_disable, cheat_time_freeze, cheat_infinite_energy
- **Teleportation**: cheat_warp (location), cheat_mine_warp (level)
- **Farming Automation**: cheat_hoe_all, cheat_water_all, cheat_grow_crops, cheat_harvest_all, cheat_plant_seeds, cheat_fertilize_all
- **Land Clearing**: cheat_clear_debris, cheat_cut_trees, cheat_mine_rocks, cheat_dig_artifacts
- **Pattern Drawing**: cheat_hoe_tiles, cheat_clear_tiles, cheat_hoe_custom_pattern (ASCII grid input)
- **Resources**: cheat_set_money, cheat_add_item (by item ID), cheat_spawn_ores
- **Social**: cheat_set_friendship, cheat_max_all_friendships, cheat_give_gift
- **Upgrades**: cheat_upgrade_backpack, cheat_upgrade_tool, cheat_upgrade_all_tools, cheat_unlock_all

---
> Source: [Hunter-Thompson/stardew-mcp](https://github.com/Hunter-Thompson/stardew-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
