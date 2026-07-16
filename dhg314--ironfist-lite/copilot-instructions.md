## ironfist-lite

> ./gradlew build                         # build mod jar (output: build/libs/)

# IronFist — AGENTS.md

## Quick start
```bash
./gradlew build                         # build mod jar (output: build/libs/)
./gradlew runClient                     # launch Minecraft client with mod
./gradlew runServer                     # launch dedicated server
./gradlew runData                       # run data generators → src/generated/resources/
```

## Build & toolchain
- **Java 17**, ForgeGradle `[6.0.16,6.2)`, official mappings, Minecraft 1.20.1 / Forge 47.4.0
- Properties expanded at build time (`${}`) in `META-INF/mods.toml` and `pack.mcmeta`
- Generated resources live in `src/generated/resources/` (currently empty)
- Gradle daemon **disabled** (`org.gradle.daemon=false`), JVM args `-Xmx3G`
- No lint, typecheck, or test commands exist

## Architecture
- **Entrypoint**: `IronFist.java` (`@Mod("ironfist")`) — registers events, config, and network
- **Player data**: stored in player's persistent NBT via `IronFistPlayer` (no capabilities)
- **Config**: Forge `ForgeConfigSpec` (common), live-synced from server to clients via custom packets
- **Network**: custom `SimpleChannel` (protocol version `"1"`) with two message types:
  - `MessageFistSync` (S2C) — sync player fist LV/XP
  - `MessageConfigSync` (C2S) — sync config changes
- **I18n**: language keys prefixed `ironfist.` — `en_us.json` and `zh_cn.json` in `assets/ironfist/lang/`
- **Commands** (all under `/ironfist`):
  - `showLV` — display fist level, XP, required XP
  - `setLV <lv>` — op level 2
  - `config <setting> <value>` — op, live-applied (see `Config.java` for all settings)

## Key conventions
- Bus subscribers use `@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD)` on the class + `@SubscribeEvent` on methods
- Deprecated APIs used intentionally: `FMLJavaModLoadingContext`, `ModLoadingContext.registerConfig` — keep suppressing with `//noinspection removal`
- `config.json` at repo root is **OpenCode agent config** (not mod config)
- Fist-only mode: when `fistOnly=true`, only empty-handed breaks grant XP; when `fistOnly=false`, any non-tool item (no `minecraft:tools` / `forge:tools` tag) also works

---
> Source: [DhG314/ironfist-lite](https://github.com/DhG314/ironfist-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
