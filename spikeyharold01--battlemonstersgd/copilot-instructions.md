## battlemonstersgd

> This document outlines the core systems (Agents) and data structures (Tools) that drive the BattleMonsters game architecture. The engine is a strict, data-driven Pathfinder 1e combat and ecology simulator built in Godot C#.

# BattleMonsters - System Agents & Tooling Guide

This document outlines the core systems (Agents) and data structures (Tools) that drive the BattleMonsters game architecture. The engine is a strict, data-driven Pathfinder 1e combat and ecology simulator built in Godot C#.

## Core Architectural Philosophy
1. **Strict Data-Driven Design:** Logic lives in generic, reusable C# scripts (`AbilityEffectComponent`) that are snapped together like Lego bricks inside Godot `Resource` files (`Ability_SO`, `StatusEffect_SO`).
2. **AI & Player Parity:** If a player can do it, the AI must know how to do it. All abilities must have a mathematical scoring mechanism (`GetAIEstimatedValue`).
3. **Phase Agnostic:** Spells and effects must function identically in both the `Arena` (Turn-based Combat) and `Travel` (Real-time Exploration/Ecology) phases.
4. **No Negative Levels:** Traditional level-drain mechanics are strictly replaced by the `CorruptionStackFramework`.

---

## 1. The Cognitive Agents (AI & Memory)

These agents are responsible for driving enemy and ally decision-making.

### `AIController` (The Brain)
*   **Purpose:** Attached to every AI creature. Generates a list of possible actions (`TurnPlan`), scores them, and executes the highest-scoring plan.
*   **Input:** Reads from `CombatMemory`, `GridManager`, and the creature's `KnownAbilities`.
*   **Output:** Triggers `Execute()` on the chosen `AIAction`.

### `AIScoringEngine` (The Calculator)
*   **Purpose:** A static utility that converts abstract tactical concepts into float scores.
*   **Input:** Takes a `TacticalTag_SO` and an `EffectContext`.
*   **Convention:** Uses `ScoreTacticalAction()` to weigh positional advantage, hazard exposure, and success probability.

### `AITacticalMatrix` (The Global Learner)
*   **Purpose:** Tracks global machine learning weights based on combat outcomes (e.g., learning that Fire doesn't work on Red Dragons).
*   **Output:** Serializes knowledge to `user://ai_tactical_matrix.json`. Alters the AI's likelihood of repeating certain actions.

### `CombatMemory` (The Short-Term Memory)
*   **Purpose:** A static whiteboard that tracks what creatures know *in the current encounter* (e.g., identified spells, revealed illusions, perceived alignments).
*   **Convention:** Always query `CombatMemory` rather than checking a target's `CreatureStats` directly to prevent AI from "cheating" with omniscient knowledge.

---

## 2. The Execution Agents (Combat & Resolution)

These static managers resolve the rules of the game.

### `CombatManager` / `CombatMagic` / `CombatAttacks`
*   **Purpose:** The central arbiters of the Pathfinder 1e ruleset. They handle attack rolls, AC calculations, saving throws, and spell resistance.
*   **Input:** Receives an attacker, defender, and the tool used (Weapon or Ability).
*   **Output:** Modifies `CreatureStats` (HP, conditions) and emits event logs.

### `StatusEffectController`
*   **Purpose:** Attached to creatures. Manages the lifecycle of buffs, debuffs, and afflictions.
*   **Convention:** Modifiers like "Fatigued" or "Sickened" do not alter base stats. Instead, systems call `GetTotalModifier()` to aggregate active status effects dynamically at runtime.

### `TurnManager`
*   **Purpose:** Controls the flow of time in the Arena phase. Calculates Initiative (including Dual Initiative) and manages Surprise Rounds.

---

## 3. The Ecological Agents (Grid, Travel, & Environment)

### `GridManager`
*   **Purpose:** The spatial truth of the world. Maps world coordinates to discrete `Node` objects containing terrain types, movement costs, and lighting data.
*   **Input:** World Vector3.
*   **Output:** Terrain profiles, Light Levels, Line of Sight blockages.

### `LineOfSightManager` / `VisibilityManager`
*   **Purpose:** Calculates Cover, Concealment, and Line of Effect.
*   **Convention:** Resolves illusions, visual obstructions (Smoke/Fog), and magical senses (Blindsight, Trueseeing).

### `ScentSystem` & `SoundSystem`
*   **Purpose:** Alternate perception channels. Creatures emit `SoundEvent` and `ScentEvent` structs which the AI uses to track invisible or hidden targets.

### `TravelPhase` & `TravelEncounterSpawner`
*   **Purpose:** Manages the real-time exploration map. Spawns ecology-validated creatures based on `BiomeTravelDefinition` weights without triggering instant combat.

---

## 4. The Blueprint Tools (Data Resources)

Content is created by assembling these Godot Resources (`.tres`).

### `Ability_SO`
*   **Purpose:** The universal container for Spells, Special Attacks, Feats, and Skill Actions.
*   **Data:** Contains Range, Cast Time, Targeting Rules, and a list of `AbilityEffectComponent`s.

### `AbilityEffectComponent`
*   **Purpose:** Modular C# scripts that execute specific logic (e.g., `DamageEffect`, `Effect_Summon`, `Effect_Silence`).
*   **Convention:** Must override `ExecuteEffect()` and `GetAIEstimatedValue()`.

### `StatusEffect_SO`
*   **Purpose:** Defines lingering effects (Buffs/Debuffs). Contains `StatModification` arrays, duration rules, and environmental protections.

### `CreatureTemplate_SO`
*   **Purpose:** The raw, read-only Bestiary data for a monster (Stats, Feats, Abilities, Immunities). Instantiated onto a `CreatureStats` node at runtime.

---

## 5. Input / Output Conventions

When writing new scripts for this architecture, adhere to the following I/O standards:

### Standardized Execution Context (`EffectContext`)
Abilities never pass raw variables. They pass an `EffectContext` object containing the entire state of the action:
```csharp
public class EffectContext
{
    public CreatureStats Caster;           // The entity performing the action
    public CreatureStats PrimaryTarget;    // The main target (if applicable)
    public Node3D TargetObject;            // Targeted world items/objects
    public Vector3 AimPoint;               // The grid location targeted
    public Godot.Collections.Array<CreatureStats> AllTargetsInAoE; // Everyone caught in the blast
    public Ability_SO Ability;             // The ability being resolved
    public bool IsMythicCast;              // Mythic tier flag
    public Resource SelectedResource;      // Sub-choices (e.g. chosen Weather)
    public Dictionary<CreatureStats, bool> LastSaveResults; // True if the target saved successfully
}
No Hardcoded Name Checks
Avoid checking if a creature's name is "Goblin". Use Data Tags:
Instead of: target.Name == "Fire Elemental"
Use: target.Template.SubTypes.Contains("Fire") or target.Template.Type == CreatureType.Elemental.
Temporary/Stateful Modifications
Never permanently alter a CreatureTemplate_SO during combat.
Stat changes must be done via a StatusEffect_SO.
Form changes (Polymorph/Wild Shape) must use a CreatureTemplateModifier_SO applied via the PolymorphController.
Asynchronous Flow
Combat resolution and movement use C# Task and async/await to allow animations, reaction popups (like Counterspells), and UI delays to process sequentially without freezing the engine thread.
Convention: Use await ToSignal(GetTree().CreateTimer(1.0f), "timeout"); for visual pacing inside Execute() methods.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Spikeyharold01)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Spikeyharold01)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
