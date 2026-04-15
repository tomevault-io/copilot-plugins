## takaro-minecraft

> This repository has an engineer skill at `.claude/skills/takaro-minecraft-engineer/`.

# Repository Engineer Skill

This repository has an engineer skill at `.claude/skills/takaro-minecraft-engineer/`.

Claude will automatically discover and use this skill. The skill contains:
- `SKILL.md` — Overview and quick reference
- `ARCHITECTURE.md` — Adapter pattern, event/action flows, key classes
- `BUILD.md` — Gradle build system, deploy scripts, version catalog
- `TESTING.md` — JUnit test commands and patterns
- `DOCKER.md` — Dev server setup, RCON, configuration
- `BOT.md` — Mineflayer test bot HTTP API and workflows
- `INTEGRATION-TESTING.md` — End-to-end testing with MCP tools, event verification, known limitations

IMPORTANT DOCUMENTATION:

https://docs.takaro.io/advanced/adding-support-for-a-new-game
https://docs.takaro.io/advanced/connection-architecture
https://docs.takaro.io/advanced/generic-connector-protocol

You must run the non-interactive version of verify instead of the normal command 

## Project Structure

Multi-platform Minecraft connector using Gradle multi-module build:

- `core/` — Pure Java, no MC dependency. WebSocket client, config, GameAdapter interface.
- `paper/` — Paper/Bukkit adapter. Shadow JAR with relocated deps.
- `neoforge/` — NeoForge adapter. ModDevGradle + Shadow.
- `fabric/` — Fabric adapter. Fabric Loom + Shadow.
- `bot/` — Mineflayer test bot with HTTP API. See `/bot` skill for usage.

## Build

Requires Java 21. Uses Gradle 9.4.0 with version catalog (`gradle/libs.versions.toml`).

```bash
./gradlew build          # Build all modules
./gradlew :paper:build   # Build single module
```

## Dev Servers

```bash
docker compose up -d paper      # Paper on :25565
docker compose up -d neoforge   # NeoForge on :25566
docker compose up -d fabric     # Fabric on :25567
docker compose up -d bot        # Test bot API on :3001
```

Deploy: `./scripts/deploy.sh paper|neoforge|fabric|all`

Debug logging: `TAKARO_DEBUG=true docker compose up -d paper` — shows raw WebSocket messages in logs.

## After Code Changes

When you modify connector code (core/, paper/, neoforge/, fabric/), you MUST:
1. Build and deploy: `/test-deploy <platform>`
2. Run relevant integration tests from INTEGRATION-TESTING.md
   - Use the Code Change → Test Mapping table
3. Use Takaro MCP tools for verification (not just logs)
4. Report results before considering the change complete

The Takaro command prefix is `+`. Confirm via `mcp__takaro__settingsGet`.

## Current Status

Full Takaro Generic Connector Protocol implemented: all 16 actions (player queries, world queries, player actions, bans, commands, teleport, kick, shutdown) and 6 game events (player connect/disconnect, chat, player death, entity killed, log).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gettakaro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gettakaro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
