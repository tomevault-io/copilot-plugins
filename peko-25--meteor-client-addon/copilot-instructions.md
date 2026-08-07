## meteor-client-addon

> - Output: `build/libs/slot-refiller-1.0.0.jar`

# Slot Refiller — AGENTS.md

## Build
```powershell
.\gradlew.bat build
```
- Output: `build/libs/slot-refiller-1.0.0.jar`
- Java 25, Gradle 9.6.1, Fabric Loom `1.16-SNAPSHOT`
- No test, lint, or CI tasks exist.

## Stack
- Minecraft 26.1.2 + Meteor Client `26.1.2-SNAPSHOT`
- Minecraft 26.1+ is non-obfuscated → **no** `mappings()` dependency; use `implementation()` not `modImplementation()`
- Entrypoint: `meteor` in fabric.mod.json, target class extends `MeteorAddon`
- Package: `com.peko25.meteoraddon`
- Mod ID: `slot-refiller` (hyphenated). Fabric Loader dep key is `fabricloader` (one word).

## API quirks (26.1.2 vs older)
| Old | 26.1.2 |
|---|---|
| `net.minecraft.item.*` | `net.minecraft.world.item.*` |
| `Inventory.getStack(int)` | `getItem(int)` |
| `Category.Player` (enum) | `Categories.Player` (class field) |
| `Item.getName()` | `getName(ItemStack.EMPTY)` |

## Module pattern
- `extends Module`, constructor passes `Categories.Player` and module ID (`"slot-refiller"`)
- `@EventHandler TickEvent.Pre` for tick logic
- `mc.player.getInventory().getItem(slot)` to read slots
- `InvUtils.move().from(src).to(dst)` for inventory clicks
- Settings via `SettingGroup`, `IntSetting`, `ItemSetting`, `BoolSetting`

---
> Source: [peko-25/meteor-client-addon](https://github.com/peko-25/meteor-client-addon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
