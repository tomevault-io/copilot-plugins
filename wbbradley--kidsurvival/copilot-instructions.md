## kidsurvival

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

```bash
./gradlew build          # Build the mod JAR (outputs to build/libs/kidsurvival-*.jar)
```

There are no tests. The mod is verified by loading it in Minecraft via PrismLauncher.

## Release Workflow ("ship it")

When asked to ship/release:
1. Bump `mod_version` in `gradle.properties` (minor version bump: e.g., 1.3.0 → 1.4.0)
2. `./gradlew build` to verify
3. Commit version bump (include COMPLETED.md if modified): `chore: bump version to X.Y.Z`
4. Push to main: `git push origin main`
5. Tag and push: `git tag vX.Y.Z && git push origin vX.Y.Z`

The tag push triggers `.github/workflows/release.yml` which builds the JAR and creates a GitHub Release with the artifact attached.

## Architecture

This is a Minecraft 1.21.11 Fabric mod (Java 21, Fabric Loom 1.13.6). Server-side only — no client entrypoint, no mixins in use.

**Two source files:**

- **`KidSurvivalMod.java`** — mod entry point (`ModInitializer`). Registers all commands (`/kid`, `/hunter`, `/hunger`, `/bench`), event handlers (damage cancellation, death prevention, tick updates, attack interception), and JSON persistence. Owns the static state sets (`kidModePlayers`, `hungerModePlayers`, `benchedPlayers`).

- **`HunterTagGame.java`** — self-contained Hunter Tag game state. Manages round lifecycle (start → tag → end → auto-restart), freeze enforcement, scoreboard teams/sidebar, glowing effects, player loadout (elytra + fireworks + boat), scoring (runner tick accumulation), and join/disconnect handling. Serializes to/from JSON for persistence.

**State persistence:** All state is saved to `kidsurvival.json` in the world save directory via `ServerLifecycleEvents.SERVER_STARTED/SERVER_STOPPING`. Format is GSON-serialized JSON with UUID arrays and nested objects.

**Tick system:** Both files maintain tick counters. Periodic actions use modulo checks: action bar updates every 20 ticks (1s), scoreboard updates every 20 ticks, glowing refresh every 60 ticks (3s).

**Key design pattern:** `HunterTagGame` receives a `Set<UUID> benchedPlayers` reference from `KidSurvivalMod` during `startRound()` and stores it as a field, avoiding parameter threading through all internal methods.

## Conventions

- Never commit or `git add` PLAN.md
- Commit messages use semantic prefixes: `feat:`, `fix:`, `chore:`, `ci:`, `docs:`
- No attributions in commit messages
- Version lives in `gradle.properties` (`mod_version`) and is substituted into `fabric.mod.json` at build time
- The Minecraft JAR with Yarn mappings is cached at `.gradle/loom-cache/` — use `javap` against it to verify API signatures when unsure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wbbradley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wbbradley)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
