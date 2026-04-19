## crisis-unleashed-app

> Defines technical implementation of core game mechanics including battlefield, cards, factions and combat resolution


# Game Algorithms

Core Game Systems:

1. Battlefield Management (Importance: 95)

- Coordinate system: axial (q, r); cube coordinates used only for distance.
- Lanes: battlefield divides into left, center, right zones based on coordinates.
- ZoC: radius 1; entering a hex adjacent to an enemy adds +Z to move cost; exiting engaged hexes is disallowed unless ability "Disengage".
- Pathfinding: A\* with h = hex_distance \* base_cost; edge cost = terrain_cost(q,r) + faction_modifier(unit, q,r); tie-breaker: lowest total cost, then lowest q, then lowest r.
- Occupancy: one unit per hex; stacking not allowed; summons follow same constraint unless flagged "ethereal".

2. Turn Sequencing (Importance: 90)

- Order: Start-of-turn triggers → Resource gain → Deploy → Action → End → Cleanup.
- Resources: energy += floor(turn/2) + faction_bonus; momentum capped at 10; overflow is lost.
- Initiative: compare unit.initiative; ties break by lane (L<C<R), then position (lowest q, then r).
- Crises: evaluated at End → before Cleanup; define trigger thresholds and once-per-turn constraint.

3. Combat Resolution (Importance: 85)

- Timing windows: Pre-attack → On-attack → On-hit → Post-hit → On-kill → End-of-combat.
- Damage application: simultaneous within the same initiative bucket; shields/guards reduce before HP.
- Death resolution: mark-dead, resolve on-death triggers, then remove from board; chain reactions processed FIFO.
- Status stacking: additive unless flagged "exclusive"; duration decrements at Cleanup.
- Faction-specific combat modifiers
- Unit ability triggers during combat
- Battlefield position effects on combat

4. Card Deployment Logic (Importance: 80)

- Position validation based on card type
- Resource cost verification (Energy/Momentum)
- Faction-specific deployment rules
- Unit type placement restrictions
- Synergy calculations with existing battlefield state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustAGhosT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
