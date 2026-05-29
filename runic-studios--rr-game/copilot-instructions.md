## rr-game

> This file documents project-wide coding conventions that all contributors (human and AI) must

# AGENTS.md: Project Conventions for AI Agents & Contributors

This file documents project-wide coding conventions that all contributors (human and AI) must
follow. When in doubt, refer here first.

---

## 1. Style Constraints:

- Favour Canadian English unless the name of a word comes from a spelling derived from Minecraft
  or an external plugin (i.e. "armor" is spelt without a "u" in Minecraft)
- Avoid using ASCII arrows (→) and instead do ones like "->"
- Always write out "exception" instead of "e" for variable names.
  - Avoid single letter variable names, unless in writing in Go.
- Do not quote this AGENTS.md file in comments when making changes, just follow the rules without quoting.
- Avoid using emdashes (—) and instead favour colons or parenthesis
- You should NOT use inline fully qualified name imports unless strictly necessary and instead favour
  writing import lines. 
  - If you have accidentally written FQNs, you can use the following regex (case sensitive) for searching for bad FQN imports:
  `^(?!\s*(?:import|package)\b).*?\b[a-z][a-z0-9_]*(?:\.[a-z][a-z0-9_]*)+\.[A-Z][A-Za-z0-9_]*(?:\.[A-Z][A-Za-z0-9_]*)*\b`
  - FQN imports are okay in javadocs if they are necessary (class wasn't imported)
---

## 2. Database Error Handling: Fail Loud

**Any failure from MongoDB (load, save, or lock) must NEVER fail silently.**

- **During login**: lock acquisition failure or document load failure: kick the player
  immediately with a descriptive error message, log at `ERROR` level.
- **Periodic save loop**: any save or lock-renewal failure: kick the player, log at `ERROR`
  level. The subsequent `PlayerQuitEvent` will attempt a final save.
- **On quit (final save / lock release)**: the player is already disconnecting, so kicking is
  not possible. Log at `ERROR` level instead. Make the log message include the player UUID and
  a human-readable description of what was being saved.

**Template:**
```kotlin
val result = repository.save(doc)
if (result.isFailure) {
    logger.error("FATAL: save failed for player $playerId", result.exceptionOrNull()!!)
    // kick if online; log-only if during logout
    Bukkit.getPlayer(playerId)?.kick(Component.text("Data save failed. Please reconnect."))
}
```

---

## 3. Thread Contexts

The game server runs on two distinct execution contexts. Using the wrong one causes race
conditions or crashes. **Every `withContext(...)` call must have a comment explaining why the
context is being switched.**

### Minecraft main thread (`plugin.minecraftDispatcher`)
Use for:
- All Bukkit API calls (player.teleport, inventory.setItem, attribute changes, etc.)
- All Bukkit event firing (`Bukkit.getPluginManager().callSuspendingEvent(...)`)
- All mutations to `GameSessionManager.sessions` and `GameSessionManager.players` maps
- All mutations to `GameSession.activeCharacterSlot`

### Async dispatcher (`plugin.asyncDispatcher`)
Use for:
- All MongoDB calls (PlayerRepository.loadOrCreate, save; PlayerLockRepository.acquireOrRenew, release)
- The periodic save loop body
- Any heavy computation that would stall the tick loop

### Data mutation (any thread, but requires the data lock)
`GameSession.document` fields may be mutated from **any** thread or coroutine, as long as the
caller holds the per-player `PlayerDataLock`. The lock is acquired automatically:

- `gamePlayer.withPlayerData { settings.tips = true }` — suspending, acquires the lock
- `gameCharacter.withCharacterData { traits.exp += gained }` — suspending, acquires the lock
- `gameCharacter.withSyncCharacterData { traits.location = ... }` — blocking (for sync event
  handlers on the MC main thread), acquires the lock via `runBlocking`

The save loop acquires the lock briefly to take a `copy()` snapshot, then releases before the
MongoDB call, so gameplay callers are never blocked for the duration of a DB write.

**Pattern:**
```kotlin
// Switch to async for the DB call
withContext(plugin.asyncDispatcher) {
    val result = playerRepository.save(doc)
    // ...
}
// Switch back to MC thread to fire events / update Bukkit state
withContext(plugin.minecraftDispatcher) {
    Bukkit.getPluginManager().callSuspendingEvent(event, plugin).joinAll()
}
```

---

## 4. Schema Migration Rules

Every structural change to `PlayerDocument` requires:

1. **Bump `PlayerDocument.CURRENT_VERSION`** by 1.
2. **Write a new `Migration` implementation** in `data/src/main/kotlin/com/runicrealms/game/data/migration/`
   that transforms the old BSON shape to the new shape.
3. **Register the migration** by adding it to the list in `MongoModule.provideMigrationChain()`.

**Additive-only changes** (new field with a default value, e.g. `val newField: Int = 0`) do NOT
require a migration as long as the old BSON simply omits the field and the default takes effect.

**Never** rename or remove a field without a migration.

---

## 5. Data Mutation Pattern

`PlayerDocument` and all nested types are Kotlin `data class`es with `var` fields. To mutate
player or character data, use the accessor functions on `GamePlayer` / `GameCharacter`. The
per-player `PlayerDataLock` is acquired automatically — never acquire or release it manually.

**From a suspending context (coroutine):**
```kotlin
// Mutate player-level data
gamePlayer.withPlayerData {
    settings.tips = true
    traits.lastLogin = Instant.now()
}

// Mutate character-level data
gameCharacter.withCharacterData {
    traits.exp += gained
    traits.level = newLevel
    inventory.items = updatedItems
}

// Mutate the full document (e.g. the characters map)
gamePlayer.withDocument {
    characters[slot]?.traits?.classType = ClassType.WARRIOR
}
```

**From a synchronous Bukkit event handler (MC main thread, non-suspending):**
```kotlin
// withSyncCharacterData uses runBlocking internally — only use from plain sync event handlers
gameCharacter.withSyncCharacterData {
    traits.location = player.location.toLocationData()
}
```

Do **not** access `gameSession` or `gameSession.document` from outside the `data` module.
Do **not** use `.copy(...)` chains to build updated objects; assign fields directly instead.

---

## 6. Holograms: FancyHolograms (replaces HolographicDisplays)

The project uses **FancyHolograms** (`de.oliver.fancyholograms`) as the hologram library.
HolographicDisplays is no longer available. Use the pattern below for all hologram creation.

### Creating a non-persistent text hologram

```kotlin
import de.oliver.fancyholograms.api.FancyHologramsPlugin
import de.oliver.fancyholograms.api.data.TextHologramData

// Unique name required - two holograms with the same name will conflict
val hologramName = "my_effect_${playerUuid}_${targetUuid}"
val hologramData = TextHologramData(hologramName, location)
hologramData.isPersistent = false  // ALWAYS false for runtime/effect holograms
hologramData.text = listOf("<gold>Effect x3</gold>")

val hologramManager = FancyHologramsPlugin.get().hologramManager
val hologram = hologramManager.create(hologramData)
hologramManager.addHologram(hologram)
hologram.refreshForPlayersInRange()
```

### Updating a hologram's text

```kotlin
hologram.hologramData.text = listOf("<red>New Text</red>")
hologram.refreshForPlayersInRange()
```

### Removing a hologram

```kotlin
hologramManager.removeHologram(hologram)
```

### Migration notes (from HolographicDisplays)

| HolographicDisplays | FancyHolograms equivalent |
|---|---|
| `HologramsAPI.createHologram(loc)` | `hologramManager.create(TextHologramData(name, loc))` + `hologramManager.addHologram(hologram)` |
| `hologram.appendTextLine(text)` | `hologramData.text = listOf(text)` (full list replace) |
| `hologram.delete()` | `hologramManager.removeHologram(hologram)` |
| `hologram.teleport(loc)` | `hologramData.location = loc; hologram.refreshForPlayersInRange()` |
| Per-player visibility | TODO: Not yet implemented - see StackHologram.kt. Requires `hologram.setVisibleByDefault(false)` + `hologram.addViewer(player)` on each player, but FancyHolograms API differs. |

### Key constraints

- **Names must be globally unique**: Use UUIDs or compound keys (e.g. `"stacks_BLEED_<casterUuid>_<recipientUuid>"`).
- **Always set `isPersistent = false`** for runtime holograms that should not survive server restarts.
- FancyHolograms text supports MiniMessage format (`<red>`, `<gold>`, `<bold>`, etc.).
- Adventure `Component` is NOT directly supported as a line - convert to MiniMessage string first.

---

## 7. Logging

- All logging within this plugin should be done through a logger created by a LoggerFactory
  whose name corresponds to the gradle module the class is located in.
- Classes should not share logger factories and each should have their own value

```
private val logger = LoggerFactory.getLogger("gameplay")
```

---
> Source: [Runic-Studios/RR-Game](https://github.com/Runic-Studios/RR-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
