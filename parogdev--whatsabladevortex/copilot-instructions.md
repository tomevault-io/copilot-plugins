## whatsabladevortex

> Tracks Blade Vortex blades via player buffs and visualizes the Minion Pact snapshot lifetime. The default HUD cannot distinguish individual BV casts or show per-blade timers -- each blade is a separate `new_new_blade_vortex` buff with its own timer. The plugin groups blades into temporal clusters, tracks the snapshot state machine (spectre sacrifice), and shows Unleash seal charges from the `anticipation` buff.

# CLAUDE.md - WhatsABladeVortex

## Purpose

Tracks Blade Vortex blades via player buffs and visualizes the Minion Pact snapshot lifetime. The default HUD cannot distinguish individual BV casts or show per-blade timers -- each blade is a separate `new_new_blade_vortex` buff with its own timer. The plugin groups blades into temporal clusters, tracks the snapshot state machine (spectre sacrifice), and shows Unleash seal charges from the `anticipation` buff.

## Key Behavior

- BV blade detection: reads all `new_new_blade_vortex` buff entries from the player's `Buffs` component. Each entry is one blade with its own `Timer` and `MaxTime`.
- Cluster grouping: blades with timers within `ClusterWindowMs` (default 150ms) are grouped into the same cast cluster. This captures Greater Unleash releases where seals + base cast fire in the same tick.
- BladeTracker: stateless per-frame rebuild from buff data using object pooling. Pre-allocates pools of 10 `BladeInfo` and 10 `BladeCluster` instances to avoid GC pressure. Sorts blades by timer descending and groups into clusters by timer gap.
- Snapshot state machine: `Ready` (waiting for spectre sacrifice) -> `Active` (spectre died with blades present) -> `Lost` (all blades expired) -> `Ready` (spectre resurrected). Transition to Active requires `_prevSpectreAlive && !spectreAlive && hasBlades`.
- Spectre tracking: alive state determined by absence of `number_of_orb_spectres` buff (buff present = dead spectre). HP read from `raise_spectre` skill's `DeployedObjects` when alive.
- Minion Pact detection: reads `SupportMinionPactSacrificeNearbyDamageableMinionToGainPermillageOfMaxLifeAsPhysicalDamage` stat from the `blade_vortex` ActorSkill. Calculates added physical damage as `spectreMaxHp * permillage / 1000`.
- Unleash seal meter: reads `anticipation` buff's `BuffCharges`. Purely informational -- not BV stacks.
- Skill icon: reads `EffectsPerLevel.SkillGemWrapper.ActiveSkill.Icon` from the `blade_vortex` ActorSkill. Cached on render state and drawn as semi-transparent background behind the blade count.
- Sound alert: plays `snapshot_lost.wav` via `GameController.SoundController.PlaySound()` when snapshot transitions to Lost. Configurable sound file and toggle in settings.
- Minion Pact zone warning: with Minion Pact, builds typically run one spectre for maximum minion life stacking. Once the spectre dies and blades expire (snapshot lost), the ONLY way to recover the snapshot is to zone (spectre revives on zone change). The compact panel shows "ZONE" in red when this state is detected.
- Render state DTO: all Tick data is packed into a `BladeVortexRenderState` object, passed to both the overlay renderer and the custom settings UI.
- AreaChange clears all tracked state. Grace period guard skips Tick processing. Plugin sends no input.

## Architecture

Follows the established plugin pattern (`/buff-tracker`, `/player-monitor` skills):

1. **Tick()** -- reads all components (Buffs, Actor, Life on deployed minions), rebuilds blade clusters, runs snapshot state machine, builds `BladeVortexRenderState` DTO. No drawing.
2. **Render()** -- delegates to `SnapshotPanelRenderer` using cached render state. No component access.
3. **DrawSettings()** -- delegates to `BladeVortexSettingsUi` for custom ImGui settings panel with live data display.

## Files

- `WhatsABladeVortex.cs` -- main plugin. Tick (buff read, cluster rebuild, state machine), Render (calls renderer + edge flash), DrawSettings, AreaChange.
- `BladeTracker.cs` -- `BladeInfo`, `BladeCluster`, `BladeTracker` with object-pooled Rebuild, FindClusterBars, FindNextLoss.
- `SnapshotPanelRenderer.cs` -- pure draw of overlay panel. Accepts `BladeVortexRenderState` DTO. DrawCompact (seals + bars), DrawFullPanel (header, clusters, footer), DrawDebugWindow.
- `BladeVortexSettingsUi.cs` -- custom ImGui settings panel with tabbed layout (Overlay, Debug), pill toggles, sliders, text inputs, sound folder button, live state status bar.
- `WhatsABladeVortexSettings.cs` -- all settings with `[Menu]` parentIndex hierarchy: Display, Position, Alerts, Sound, Debug.

## Gotchas

- `Color` defaults to `System.Drawing.Color` when `using SharpDX;` is present -- file must `using Color = SharpDX.Color;`.
- `ExileCore.Graphics` collides with `System.Drawing.Graphics`; renderer does `using Graphics = ExileCore.Graphics;`.
- `Buff.Charges` is obsolete; use `Buff.BuffCharges`.
- Do NOT treat the `anticipation` buff charges as a blade count -- it is Unleash seals.
- Spectre alive detection uses buff absence (`number_of_orb_spectres` not present = alive). `DeployedObjects.Entity.IsAlive` is unreliable.
- `Life.MaxHP` returns 1 when the component address is null; only update cached HP when value > 1.

## Verification

1. Restart the HUD after building (no auto-recompile).
2. Open plugin settings panel -- should show tabbed UI with Overlay/Debug tabs, live status bar, and Sound section.
3. Enter a map with at least one Raise Spectre minion alive.
4. Cast BV once -> compact panel should show centered seal pips, BV icon with total blade count, and timer bar.
5. Let blades expire -> panel shows LOST text, plays sound alert. If Minion Pact + spectre dead, shows "ZONE" warning.
6. Settings tab live data should update in real-time (blade count, seal count, spectre status, Minion Pact info).
7. "Open Folder" button in Sound section should open PoeHelper/Sounds/ in Explorer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ParogDev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
