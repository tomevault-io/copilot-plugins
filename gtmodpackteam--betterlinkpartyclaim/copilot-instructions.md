## betterlinkpartyclaim

> BLPC — Minecraft 1.12.2 Forge mod. Chunk claiming with party-based sharing. Optional BetterQuesting integration.

# CLAUDE.md

BLPC — Minecraft 1.12.2 Forge mod. Chunk claiming with party-based sharing. Optional BetterQuesting integration.

## Build

RetroFuturaGradle (RFG) + GTNH Buildscripts. **Do not edit `build.gradle`** (auto-generated). Config: `buildscript.properties`.

```bash
./gradlew build              # Full build (includes spotlessCheck)
./gradlew runClient          # Launch Minecraft client with the mod
./gradlew runServer          # Launch Minecraft server with the mod
./gradlew spotlessApply      # Auto-format code (run before committing)
./gradlew spotlessCheck      # Check formatting without fixing
./gradlew test               # Run JUnit 5 tests
```

## Key Rules

- **Java 17 syntax mandatory** (Jabel → JVM 8): switch expressions (`->`), pattern matching `instanceof`, `var` for obvious types. Details in `.claude/skills/blpc-overview/SKILL.md`.
- **Imports**: Always use `import` statements, not FQCN. Spotless enforces ordering.
- **Network messages**: Wire protocol IDs are stable. C→S party operations multiplex through `MessagePartyAction` (discriminator: `int action`). S→C client toasts/notifications multiplex through `MessageClientNotify` (discriminator: `int kind`). To add a new operation/notification, append a new `ACTION_*` / `KIND_*` constant — do **not** add a new top-level wire ID. Top-level IDs are only added for genuinely new message families (new sync stream, new C→S request shape) and must be appended to **both** `ModNetwork.CLIENT_BOUND_MESSAGES` (server-side NoOp registration) **and** `ClientPacketHandlers.installAll()` (client-side handler registration) in the **same order** — never insert into existing positions.
- **Side boundary**: Server-side handlers live in `common/network/`; client-side (S→C) handlers live in `client/network/` with `@SideOnly(Side.CLIENT)`. IMessage classes stay in `common/network/` and must not reference any `@SideOnly` types in their own bytecode.

## Architecture

See `.claude/skills/blpc-overview/SKILL.md` for full reference (package layout, conventions, data schemas, UI patterns, config, etc.).

---
> Source: [GTModpackTeam/BetterLinkPartyClaim](https://github.com/GTModpackTeam/BetterLinkPartyClaim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
