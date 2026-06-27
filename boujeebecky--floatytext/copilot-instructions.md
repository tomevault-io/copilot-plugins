## floatytext

> Dalamud FFXIV plugin. Replaces the game's default floating combat text with WoW-style flashy numbers.

# FloatyText — Developer Notes

Dalamud FFXIV plugin. Replaces the game's default floating combat text with WoW-style flashy numbers.

## Build

```powershell
cd D:\Projects\FloatyText
dotnet build
# Output: bin\Debug\net10.0-windows\FloatyText.dll
# Load in-game: Dalamud Settings → Experimental → Dev Plugin Locations → add the DLL path
# Reload after rebuild: /xlplugins → disable/re-enable, or /xldev → reload
```

## Project structure

| File | Role |
|---|---|
| `Plugin.cs` | Entry point. Service injection, WindowSystem, Framework.Update hook. Creates FontManager, FloatyTextRenderer, CombatEventHandler. |
| `CombatEventHandler.cs` | Two spawn paths. **Instant mode** (default): spawns at packet time from `ActionEffectHook.EffectReceived` — classifies from packet flags, looks up action name/icon via Lumina `Action` sheet, filters to own/pet actions + things targeting the player, remembers spawned values; `FlyTextCreated` then suppresses the game's delayed duplicate by value-match (3s window). Unmatched fly text (DoT/HoT ticks, misses — ActorControl, not ActionEffect) spawns via the classic path. **Classic mode**: packet effects queue locally; `FlyTextCreated` dequeues for position/filter and spawns (synced to hit animations). |
| `ActionEffectHook.cs` | Unsafe hook via `FFXIVClientStructs.ActionEffectHandler.Receive`. Fires `EffectReceived(PendingEffect)` at network-packet time, once per damage/heal entry per target (iterates AoE targets via the effectTrail ulong array; 8 effect entries per target). EffectTypes 3/5/6 = damage (5 blocked, 6 parried), 4 = heal. Decodes >65535 values: `Value + (Param2 << 16)`. All pointers null-checked before deref (AVs in unsafe code are NOT catchable). |
| `ScreenPositionResolver.cs` | Projects world → screen. Uses **local player's Y** (not the enemy's) so text stays at eye-level even for raid bosses that fill the screen. Uses `IObjectTable.SearchByEntityId(uint)` (NOT `SearchById(ulong)`) for packet-level 32-bit entity IDs. |
| `FloatyTextRenderer.cs` | Manages active entries, draws via `ImGui.GetBackgroundDrawList()`. One font handle per family (large pre-baked size); renders user-specified size as a downscale → crisp. Draw order: shadow → crit glow → outline → main text → ability icon → ability name. Entry list guarded by `_entriesLock` — Spawn/Tick run on the framework thread, Draw on the render thread. |
| `FloatingTextEntry.cs` | Per-number animation state. Ease-out quadratic float, overshoot-bounce crit pop (2.0× → 1.12× → 1.0×), decaying sinusoidal shake for crits. Exposes `DisplayPosition = Position + ShakeOffset`. |
| `FontManager.cs` | Holds one `IFontHandle` per game-font family, loaded via `FontAtlas.NewGameFontHandle`. Large source size → user sizes are downscales → no bitmap blur. |
| `Configuration.cs` | `IPluginConfiguration`. Saved by Dalamud automatically. |
| `ConfigWindow.cs` | Extends `Dalamud.Interface.Windowing.Window`. Registered with `WindowSystem`. Tabbed layout (`ImGui.BeginTabBar`): Visibility / Colors / Text & Font / Animation, Save buttons below the tab bar. All ImGui property edits use local-copy pattern (can't `ref` auto-properties). |

## Dalamud v15 API notes (important)

| Topic | Detail |
|---|---|
| Target framework | `net10.0-windows` — Dalamud 15 is built on .NET 10 |
| ImGui namespace | `Dalamud.Bindings.ImGui` (was `ImGuiNET`). Reference `Dalamud.Bindings.ImGui.dll`, NOT `ImGui.NET.dll` |
| `Plugin.Name` | Removed — plugin name comes from the manifest (`FloatyText.json`) |
| `IFlyTextGui` | Inject via `[PluginService]` — `PluginInterface.Create<>()` is gone |
| `IClientState.LocalPlayer` | **Gone**. Local player is now `IObjectTable.LocalPlayer` |
| `IObjectTable.SearchByEntityId(uint)` | Use this for 32-bit packet entity IDs. `SearchById(ulong)` is a different overload for 64-bit game object IDs. |
| `FlyTextKind` enum | **Completely renamed** in v15. Old→New: `AutoAttack`→`Damage`, `CriticalHit`→`DamageCrit`, `DirectHit`→`DamageDh`, `CriticalDirectHit`→`DamageCritDh`, `NamedAttack`→`Named`, `Healing`→`Healing` (same), `CriticalHealingBuff`→`HealingCrit`, `Miss`→`Miss` (same). Also new: `AutoAttackOrDot`, `AutoAttackOrDotCrit`, `AutoAttackOrDotDh`, `AutoAttackOrDotCritDh`, `Dodge`. |
| `BattleNpcSubKind` enum | Renamed in v15: `Enemy`→`Combatant`, `Chocobo`→`RaceChocobo`. Full set: `None, BNpcPart, Pet, Buddy, Player, Combatant, RaceChocobo, LovmMinion, NpcPartyMember`. `StatusFlags` (`PartyMember`, `AllianceMember`, `Hostile`, …) unchanged. |
| `ITextureProvider` icons | `GetFromGameIcon(new GameIconLookup(iconId))` → `ISharedImmediateTexture` → `.GetWrapOrDefault()` (null until loaded — skip that frame). Cached by Dalamud, safe to call per-frame at draw time. Draw via `drawList.AddImage(wrap.Handle, min, max, uv0, uv1, tint)`. `IDalamudTextureWrap` is in `Dalamud.Interface.Textures.TextureWraps`. **Gotchas:** FlyText sends `uint.MaxValue` (-1) as well as 0 for "no icon", and `GetFromGameIcon` THROWS `IconNotFoundException` for invalid IDs (it doesn't return null) — normalize the sentinel and try/catch the lookup. |
| `MathF.Clamp / MathF.Lerp` | **Removed from .NET 10**. Use `float.Clamp` / `float.Lerp` instead. |
| `MathF` bare name | FFXIVClientStructs defines a type named `MathF` that shadows `System.MathF`. Always qualify as `System.MathF.Sin(...)` etc. |
| `WindowSystem` | Required for windows in v15. `ConfigWindow` extends `Dalamud.Interface.Windowing.Window`. Register with `_windowSystem.AddWindow()`, draw via `_windowSystem.Draw()`, dispose via `_windowSystem.RemoveAllWindows()`. |
| Font API | `PluginInterface.UiBuilder.FontAtlas.NewGameFontHandle(new GameFontStyle(GameFontFamilyAndSize.X))`. Use `handle.Available` before `handle.Lock()`. `locked.ImFont` gives `ImFontPtr`. Draw with `drawList.AddText(font, fontSize, topLeft, color, text)`. |
| `ImplicitUsings` | Must be `enable` in csproj — without it BCL types (`List<>`, `IDisposable`, etc.) fail because referenced DLLs bring in conflicting `System.Runtime` versions. |

## Features implemented

- **WoW-style rendering**: black outline (8-directional 1px), drop shadow, crit glow bloom
- **Crit pop**: overshoot-bounce scale (2.0× → 1.12× → 1.0×) over 0.25s
- **Crit shake**: decaying sinusoidal wiggle (38Hz X, 29Hz Y, 5px amplitude) for 0.35s
- **Ease-out float**: quadratic ease-out, position driven from `_startPos` (no drift accumulation)
- **Game font families**: Axis, TrumpGothic (default, WoW-like), Jupiter, Meidinger — all pre-rasterized, downscaled for crispness
- **User font sizes**: Normal (default 22px) and Crit (default 34px), sliders up to 100px. TrumpGothic source is 68px — default 2× crit pop peaks at exactly 68px (crisp).
- **Number format**: abbreviated (31k / 1.2M) or full with commas (31,249) — checkbox toggle
- **Stack spreading**: new entries nudge horizontally if they'd overlap an existing one
- **Position**: uses local player Y + 2.3 world units, enemy X/Z — text stays at eye-level for any enemy size
- **Category colours**: Normal (white), Crit (gold), Direct (cyan), CritDirect (orange-red), Healing (green), DoT (purple), Miss (grey)
- **Instant mode** (`InstantMode`, default true): numbers spawn at ActionEffect packet time (parser timing) instead of waiting for the game's delayed fly text (~0.5–1s application delay). Toggle on the Animation tab. Falls back to fly-text timing automatically if the hook is down.
- **Visibility toggles**: per-type (damage, heal, DoT, miss) and per-target (self, party, enemies)
- **Target filtering (wired)**: target relation resolved from ActionEffectHook's target entity ID → object table. Self = local player EntityId; Party = `StatusFlags.PartyMember`/`AllianceMember` players + `Pet`/`Buddy`/`NpcPartyMember` NPCs; Enemy = `Combatant`/`BNpcPart`. Unknown relation (hook down, DoT ticks, AoE overflow) **always shows** — never silently drops text. Filtered-out text still sets `handled = true` (vanilla stays suppressed).
- **Ability name + icon**: action name from `text1` drawn below the number (white, outlined, size = `AbilityNameScale` × number size); action icon from the `icon` param drawn left of the number, square (side = `AbilityIconScale` × text height), scales with crit pop, fades with opacity. Toggles: `ShowAbilityName` / `ShowAbilityIcon`; size sliders on the Text & Font tab. Auto-attacks have neither (empty `text1`, icon 0) — number only.

## Known issues / future work

- **ActionEffectHook**: Uses `FFXIVClientStructs.ActionEffectHandler.MemberFunctionPointers.Receive`. Can fail silently on patch day. Plugin gracefully falls back to target-object position when hook is inactive. Text still spawns via FlyTextCreated even if hook fails.
- **DoT ticks vs auto-attacks**: Dalamud v15 merged these into `AutoAttackOrDot*` — impossible to distinguish without deeper packet parsing.
- **Target filtering accuracy (classic mode)**: depends on the packet queue staying in sync with FlyTextCreated. DoT ticks arrive via ActorControl → relation Unknown → always shown. Unknown-relation text intentionally bypasses the filter.
- **Instant-mode dedup is value-based**: a DoT tick whose value coincidentally equals an instant-spawned number within 3s gets wrongly suppressed (rare). Damage taken in instant mode classifies physical/magical only via packet flags, so it's all gated on `ShowPhysicalDamage`. Target-filtered instant effects still call `RememberInstant` so their delayed fly-text duplicate is suppressed rather than falling through to the classic path at a wrong fallback position.
- **Type toggles don't suppress vanilla** (intentional — decided 2026-06-09): with e.g. ShowHealing off, `ClassifyKind` returns null → handler returns without `handled = true` → the game's own heal numbers show. Owner's call: our toggles control OUR text only; players who want vanilla text gone should use the game's own options panel. Don't change unless user feedback asks for it.
- **Backups**: dated source zips in `Backups\` (not a git repo yet — owner deferring until more features land).
- **Instant-mode source filter**: only own actions, own pet's (`IBattleNpc.OwnerId`), and effects targeting the player spawn instantly. Party members' damage on enemies still shows via the delayed FlyTextCreated path (when the game creates fly text for it).

## Configuration properties

```
InstantMode         default true  (spawn at packet time vs game fly-text timing)
ShowPhysicalDamage / ShowMagicalDamage / ShowHealing / ShowDoTTicks / ShowMisses
ShowOnSelf / ShowOnParty / ShowOnEnemies
ShowAbilityName     default true  (action name below number)
ShowAbilityIcon     default true  (action icon left of number)
AbilityNameScale    default 0.45f (name size as fraction of number size, slider 0.2–1.0)
AbilityIconScale    default 0.75f (icon side as fraction of text height, slider 0.3–1.5)
ColorNormalHit / ColorCritHit / ColorDirectHit / ColorCritDirectHit
ColorHealing / ColorDoTTick / ColorMiss
FontChoice          (0=Axis, 1=TrumpGothic, 2=Jupiter, 3=Meidinger)
FontSizeNormal      default 22f
FontSizeCrit        default 34f
HoldDuration        default 0.5s
FadeDuration        default 0.8s
FloatSpeed          default 60 px/s (average; ease-out, so initial speed is 2×)
AnimationStyle      (0=straight, 1=arc left, 2=arc right, 3=random)
CritPopAnimation    default true
CritShake           default true
CritGlowEffect      default true
OutlineEnabled      default true
ShadowEnabled       default true
AbbreviateNumbers   default true
```

---
> Source: [BoujeeBecky/FloatyText](https://github.com/BoujeeBecky/FloatyText) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
