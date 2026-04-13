## muellereuchre

> When working with complex files.


# JSDoc Best Practices for the Euchre Project

This ruleset outlines the mandatory JSDoc techniques for documenting the Euchre project. Proper documentation is critical for maintaining Layer 1 purity, enabling test-driven development with `node:test`, and ensuring clarity across the layered architecture. Consistent and detailed JSDoc provides a structured format that is easily understood by both developers and AI assistants, aligning with the project's development standards.

### Table of Contents

1. [Basic Function Documentation (`@param`, `@returns`)](#1-basic-function-documentation-param-returns)
2. [Defining Complex Objects with In-Place `@typedef`](#2-defining-complex-objects-with-in-place-typedef)
3. [Documenting Dependency Injection for Tests](#3-documenting-dependency-injection-for-tests)
4. [Creating Types from Constants (`@typedef`, `keyof typeof`)](#4-creating-types-from-constants-typedef-keyof-typeof)
5. [Mandatory Reference Tracking with `@see`](#5-mandatory-reference-tracking-with-see)

---

### 1. Basic Function Documentation (`@param`, `@returns`)

This is the foundational technique for describing a function's contract. For this project, it's essential for documenting pure Layer 1 functions that operate on game state.

**The Technique:**
Use `@param {type} name - Description` for each parameter and `@returns {type} - Description` for the return value. All Layer 1 functions that transform state must document their inputs and outputs.

**Example (Based on `constants-import-usage-guide.md`):**

```javascript
import {
  GAME_PHASES,
  PLAYER_POSITIONS,
  CARD_VALUES,
  CARD_SUITS,
  TEAMS,
} from "@/config/constants";

/**
 * @typedef {import('./jsdoc.md').GamePhase} GamePhase
 * @typedef {import('./jsdoc.md').PlayerRole} PlayerRole
 * @typedef {import('./jsdoc.md').Card} Card
 * @typedef {import('./jsdoc.md').SuitConstant} SuitConstant
 */

/**
 * Creates the initial state object for a new game. This is a pure function.
 *
 * @param {{ team: string }} player - The player object, containing their team.
 * @returns {{phase: GamePhase, dealer: PlayerRole, cards: Card[], trump: SuitConstant}} A new state fragment.
 */
function setupInitialState(player) {
  // Access the prefixed property from the imported `TEAMS` object.
  if (player.team === TEAMS.TEAM_NS) {
    // ... team logic
  }

  // Prefixed constants are explicit and unambiguous.
  return {
    phase: GAME_PHASES.GAME_PHASE_LOBBY,
    dealer: PLAYER_POSITIONS.PLAYER_SOUTH,
    cards: CARD_VALUES,
    trump: CARD_SUITS.CARD_SUIT_HEARTS,
  };
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SevWren) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
