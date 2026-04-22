## robomage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- **Build the project**: `make clean && make`
- **Clean build artifacts**: `make clean`
- **Build for release**: `make BUILD=RELEASE`
- **Disable GUI**: `make HEADLESS=TRUE`

The compiled binary is output to `bin/robomage`.

## Testing guidelines
-Non fatal errors are not acceptable
-Draws are not acceptable
-train.py --diag and train.py --watch-scripted are helpful for checking new builds; 
--when testing with diag and --watch-scripted, supply --deck and --opponent arguments to test cards/decks relevant to recently implemented features

### Test harness for card behavior verification

`train/test_harness.py` runs the engine with `--machine --narrative --no-shuffle` so that deck file order = draw order (first 7 cards become the starting hand), and game narrative is visible alongside decoded binary state.

**Engine flags used by the harness:**
- `--no-shuffle` — skip initial library shuffle; cards are drawn in deck file order
- `--narrative` — enable game_log output in machine mode (perfect information)

**Quick start — specify hands inline:**
```bash
train/.venv/bin/python train/test_harness.py \
  --hand-a "Mountain,Lightning Bolt" \
  --library-a "Island,Island,Mountain,Mountain,Mountain,Mountain,Mountain,Mountain" \
  --hand-b "Forest,Grizzly Bears" \
  --library-b "Forest,Forest,Forest,Forest,Forest,Forest,Forest,Forest" \
  --scripted --max-decisions 30
```

**Using pre-made deck files** (for precise library sizes without auto-padding):
Write `.dk` files to `bin/resources/decks/temp/` with `1 CardName` per line (hand cards first), then pass the deck name relative to `decks/`:
```bash
train/.venv/bin/python train/test_harness.py \
  --deck-a temp/my_test_a --deck-b temp/my_test_b --scripted
```

**Play modes:**
- `--scripted` — rule-based agent (from env.py) plays both sides automatically
- `--actions "9,0,7,0,8"` — pre-scripted action index sequence
- `--interactive` — prompt for each decision (useful for step-by-step debugging)
- Default (no flag) — auto-passes every decision

**JSON scenario files:**
```bash
train/.venv/bin/python train/test_harness.py --scenario scenario.json
```
```json
{
  "name": "bolt_kills_bear",
  "hand_a": ["Mountain", "Lightning Bolt"],
  "library_a": ["Island", "Island", "Mountain", "Mountain", "Mountain", "Mountain", "Mountain", "Mountain"],
  "hand_b": ["Forest", "Grizzly Bears"],
  "library_b": ["Forest", "Forest", "Forest", "Forest", "Forest", "Forest", "Forest", "Forest"],
  "actions": [9, 0, 7, 0, 8],
  "seed": 1
}
```

**Output format** — at each decision point the harness prints:
- Narrative lines from the engine (casts, damage, zone changes, combat)
- Decoded game state (life, mana, hand contents, battlefield permanents with P/T/status, stack, graveyards)
- Available actions with human-readable descriptions (e.g. "Cast Lightning Bolt", "Target Grizzly Bears (opp)")

**Notes:**
- `--hand-a`/`--hand-b` auto-pads the library to a minimum 15-card deck. For precise small libraries (e.g. testing Thassa's Oracle with near-empty deck), write deck files manually to `decks/temp/` and use `--deck-a`/`--deck-b` instead.
- Card names in deck files omit apostrophes: `Thassas Oracle`, `Lions Eye Diamond`
- The scripted agent always keeps (no mulligan), casts spells when affordable, plays lands, and attacks with all creatures
- Temp deck files in `decks/temp/` are cleaned up automatically when using `--hand-a`/`--hand-b`; manually created files in `decks/temp/` are not

## Code Style

- Don't put new functions in main.cpp
- Don't edit my comments for spelling or punctuation. Only change them if something substantive changed.
- Avoid inline logic for anything that will be repeated; write new functions that are reusable
- Declare local functions as private in the class, if the header contains a single class/struct, if header does not contain a class, write them as static functions in global namespace C-style.
- Iterate through mEntities when possible (working within a system class), rather than iterating through all entities
- Try to consolidate iterations through entities within a function, rather than iterating through many times
- Static (local) functions should be forward declared at top of source file for clarity
- C++17 with exceptions disabled (`-fno-exceptions`)
- GUI is written in C99 with raylib
- Uses clang-format configuration in `.clang-format`

## Project Overview

Robomage is a C++ implementation of a Magic: The Gathering game engine using an Entity Component System (ECS) architecture.

Every game decision logged as an integer. Games can be replayed deterministically when provided with the correct seed.

The python side of the project enables machine learning of the game and analysis.

## Architecture

### ECS Pattern

The codebase follows an Entity Component System architecture based on [Austin Morlan's ECS tutorial](https://austinmorlan.com/posts/entity_component_system/):

- **Entities** (`src/ecs/entity.h`): Simple uint32_t IDs, maximum 5000 entities
- **Components** (`src/components/`): Pure data structs attached to entities
- **Systems** (`src/systems/`): Logic that operates on entities with specific component signatures
- **Coordinator** (`src/ecs/coordinator.h`): Central manager accessed via `global_coordinator` singleton

### Component Types

- **CardData**: Base card information (name, types, mana cost, oracle text, power/toughness, ability templates)
- **Zone**: Location tracking (library, hand, battlefield, stack, graveyard, exile, sideboard) with ownership and distance_from_top
- **Permanent**: Added when a card enters the battlefield; holds controller, tapped state, summoning sickness, and activated ability list
- **Ability**: Triggered, activated, or spell abilities with source/target/amount; also used as standalone stack entities for activated abilities
- **Creature**: Power/toughness, attacking/blocking state (added alongside Permanent for creatures)
- **Damage**: Damage counter tracking for creatures
- **Spell**: Marks a card entity that is currently on the stack as a spell
- **Effect**: Continuous effects (framework present, not yet applied)
- **Token**: Token permanent data (name, type)
- **Player**: Life total, mana pool, lands played this turn

### System Types

- **Orderer** (`src/systems/orderer.h`): Zone operations — card movement, drawing, shuffling, stack ordering
- **StateManager** (`src/systems/state_manager.h`): State-based effects (lethal damage, player death), permanent component lifecycle, `determine_legal_actions`
- **StackManager** (`src/systems/stack_manager.h`): Stack resolution — spells resolve to battlefield or graveyard; standalone ability entities resolve via `Ability::resolve()` then are destroyed

### Game Flow

The `Game` struct (`src/classes/game.h`) tracks:
- Current turn/step (UNTAP, UPKEEP, DRAW, FIRST_MAIN, BEGIN_COMBAT, DECLARE_ATTACKERS, DECLARE_BLOCKERS, FIRST_STRIKE_DAMAGE, COMBAT_DAMAGE, END_OF_COMBAT, SECOND_MAIN, END_STEP, CLEANUP)
- Active player and turn order
- Timestamp for ordering simultaneous events
- RNG seed and generator for reproducibility
- Delayed triggers (fire on specific future game events)
- Action history ring buffer (last 15 actions, used in ML observation)

Game loop in `src/main.cpp`:
1. State-based effects check (lethal damage, player death, permanent lifecycle, mandatory choices)
2. Mandatory choices (declare attackers, declare blockers, cleanup discard) handled via `proc_mandatory_choice`
3. Priority check and step advancement — if both players pass, advance step or resolve top of stack
4. Determine and display legal actions
5. Read player input via `InputLogger` (CLI, replay, or machine mode)
6. Execute chosen action via `process_action`

### Ability System

Ability categories resolved by `Ability::resolve()` in `src/components/ability.cpp`:
- `"AddMana"` — mana ability; handled at activation, never goes on the stack
- `"ChangeZone"` — zone search (e.g. fetch lands); prompts player to search, then moves card
- `"DealDamage"` — deals `amount` damage to `target` (player or creature)
- `"Destroy"` — moves `target` from battlefield to graveyard (checks target still on battlefield)
- `"Draw"` — draw cards
- `"Mill"` — put cards from library to graveyard
- `"Pump"` — add +X/+Y to a permanent's power/toughness
- `"Counter"` / `"PutCounter"` — put a counter on target permanent
- `"Token"` — create token permanents
- `"Attach"` — attach equipment or aura to target
- `"Untap"` — untap a permanent
- `"Phases"` — phase out target permanent
- `"Dig"` — look at top N cards, choose one matching filter; rest go to bottom
- `"Surveil"` / `"RearrangeTopOfLibrary"` / `"PeekAndReveal"` — look at and arrange top of library
- `"SylvanLibrary"` — draw 2, then choose: pay 4 life each or put on top
- `"DelayedTrigger"` — register ability to fire on a future game event
- `"ExaltedBonus"` / `"ProwessBonus"` — grant combat bonuses based on keyword count

Activated abilities with `valid_tgts != "N_A"` have their target selected before costs are paid and before the ability entity is pushed onto the stack. Target legality is re-verified at resolution.

### Card Parser (`src/parse.cpp`)

Parses `.txt` card scripts from `bin/resources/cardsfolder/`. Key script fields:

**Top-level card fields:**

| Field | Notes |
|---|---|
| `Name` | Card name |
| `ManaCost` | Mana cost string; `X` flag detected automatically |
| `Types` | Space-separated type line |
| `Oracle` | Oracle text; `\n` expanded to newlines |
| `PT` | Power/toughness as `P/T` |
| `A` | Activated or spell ability line (`AB$` / `SP$`) |
| `T` | Triggered ability line |
| `S` | Static ability or alternate cost line |
| `K` | Keyword list (Delve, Prowess, Equip, etc.) |
| `R` | Replacement effect line |
| `SVar` | Named variable substitution used in ability params |

**Ability parameter fields (within A/T lines):**

| Field | Notes |
|---|---|
| `AB$ <category>` | Activated ability; `Mana` normalized to `AddMana` |
| `SP$ <category>` | Spell ability |
| `Cost$` | Activation cost: `T` = tap, `PayLife<N>`, `Sac<qty/spec>`, `Return<qty/type>` |
| `Produced$` | Mana color for `AddMana`: `W/U/B/R/G/C`, `Any`, or `Combo W U G` |
| `ValidTgts$` | Target spec: `Any`, `Player`, `Creature`, `Land`, `Land.nonBasic`, combinations |
| `NumDmg$` | Damage amount for `DealDamage` |
| `ChangeType$` | Comma-separated subtypes to search (for `ChangeZone`) |
| `Origin$` | Source zone: `Library`, `Hand`, `Graveyard`, `Exile`, `Stack` |
| `Destination$` | Destination zone: `Battlefield`, `Library`, `Hand`, `Graveyard`, `Exile` |
| `Amount$` / `NumCards$` | Numeric amount or SVar reference |
| `TokenScript$` | Token name to create (for `Token` abilities) |
| `CounterType$` | Counter type string |
| `CounterNum$` | Counter count |
| `DigNum$` | Number of cards to look at (for `Dig`) |
| `Mandatory$` | `True` if the ability is mandatory |
| `SubAbility$` | SVar reference for chained sub-ability |
| `ActivationLimit$` | Max activations per turn |

Basic lands (Mountain, Forest, etc.) get their mana ability injected by `StateManager::apply_land_abilities` based on land subtypes, not from the script.

### Card Loading System

Cards are loaded on-demand from `bin/resources/cardsfolder/`:
- Card database (`src/card_db.h`): Maps card names to entity IDs, loads scripts on first access
- Parser (`src/parse.h` / `src/parse.cpp`): Parses card scripts into ECS entities with components
- Name to UID conversion: lowercase, spaces to underscores, other characters removed

### Adding a New Card

When implementing a new card, **both** of the following steps are required:

1. Add the card to `src/card_vocab.h` — append a `{"Card Name", N}` entry where N is the next available index. `N_CARD_TYPES` in `src/machine_io.h` must be >= (highest index + 1).
2. Regenerate `train/card_costs.py` by running from the repo root:
   ```
   train/.venv/bin/python train/gen_card_costs.py
   ```
   This writes the cast-cost feature matrix used by the RL environment and extractor.

### Deck Format

Deck files (`.dk`) in `bin/resources/decks/`:
```
<quantity> <card name>
...
SIDEBOARD:
<quantity> <card name>
```

## Key Globals

- `global_coordinator`: The ECS coordinator singleton
- `cur_game`: Current game state
- `RESOURCE_DIR`: Path to resources directory (set at runtime via `getcwd`)
- `card_db`: Card name to entity ID mapping

## Reinforcement Learning

The `train/` directory contains a Python gymnasium wrapper and PPO training script.

Python venv: `train/.venv/` — activate with `source train/.venv/bin/activate` or invoke directly via `train/.venv/bin/python`.

Dependencies: `gymnasium`, `stable-baselines3`, `sb3-contrib` (for `MaskablePPO` with action masking).

### Training commands (run from repo root)

```bash
train/.venv/bin/python train/train.py                                             # train from scratch
train/.venv/bin/python train/train.py --load checkpoints/robomage_final.zip      # resume
train/.venv/bin/python train/train.py --baseline checkpoints/robomage_final.zip  # win rate vs scripted
train/.venv/bin/python train/train.py --observe checkpoints/robomage_final.zip   # watch one game
train/.venv/bin/python train/train.py --self-play                                 # self-play training
train/.venv/bin/python train/train.py --diag                                      # verify env (10 quick games)
train/.venv/bin/python train/train.py --watch-scripted                            # watch scripted vs scripted
train/.venv/bin/python train/train.py --diag --bo3                                # verify bo3 env
train/.venv/bin/python train/train.py --watch-scripted --bo3                      # watch bo3 match
```

### Best-of-three mode

`--bo3` flag (C++ and Python) runs a best-of-three match in a single process:
- Player A goes first in game 1; loser goes first in subsequent games
- Decks swap between players after each game (Player A pilots B's deck and vice versa)
- Between games both players can sideboard (swap cards between main deck and sideboard)
- Match ends when either player reaches 2 wins

**C++ output protocol (machine mode):**
- `GAME_RESULT: N Player A wins` / `GAME_RESULT: N Player B wins` — after each game
- `MATCH_RESULT: Player A wins X-Y` — terminal signal for the match

**Reward structure (from Player A perspective):**
- Individual game win/loss: +0.3 / -0.3 (intermediate)
- Match win/loss: +1.0 / -1.0 (terminal)

**State vector match context** (last 4 floats, indices 32551-32554, all 0.0 in single-game mode):
- `game_number / 3.0`, `self_match_wins / 2.0`, `opp_match_wins / 2.0`, `is_sideboard_phase`

### Machine mode protocol

`--machine` flag makes the game communicate over stdio for RL training:
- Game emits a `BQUERY` line on stdout at each decision point, followed by a binary payload
- Driver writes a single integer back on stdin
- All non-BQUERY stdout lines are game narrative and can be ignored

**BQUERY format:**
```
BQUERY: <N>\n
[float32 × STATE_SIZE  — state vector]
[int32   × MAX_ACTIONS — action categories (padded)]
[float32 × MAX_ACTIONS — action card IDs (padded)]
[float32 × MAX_ACTIONS — action controller_is_self flags (padded)]
```
- `N` = number of legal choices
- State vector: 32555 floats (see `src/machine_io.h` for layout)
- Action categories: ActionCategory enum integers (0–26)
- Card IDs: `card_vocab_index / N_CARD_TYPES`, or `-1.0 / N_CARD_TYPES` (-0.0078125) as null sentinel
- Controller flags: `1.0` = self-controlled, `0.0` = opponent, null sentinel for non-entity actions

**ActionCategory values** (emitted per legal action):

| Value | Name | Meaning |
|---|---|---|
| 0 | PASS_PRIORITY | Pass priority |
| 2 | SELECT_ATTACKER | Choose a creature to attack with |
| 3 | CONFIRM_ATTACKERS | Confirm attacker declaration (sent as -1) |
| 4 | SELECT_BLOCKER | Choose a creature to block with |
| 5 | CONFIRM_BLOCKERS | Confirm blocker declaration (sent as -1) |
| 6 | ACTIVATE_ABILITY | Activate a non-mana ability |
| 7 | CAST_SPELL | Cast a spell from hand |
| 8 | SELECT_TARGET | Choose a target for a spell/ability |
| 9 | PLAY_LAND | Play a land from hand |
| 10 | OTHER_CHOICE | Generic choice (e.g. Sylvan Library pay/return, unless costs) |
| 11 | MULLIGAN | Keep (0) or mulligan (1) |
| 12 | BOTTOM_DECK_CARD | Choose card to put on library bottom (post-mulligan) |
| 13–18 | MANA_W/U/B/R/G/C | Tap a land for the corresponding color |
| 19 | SEARCH_LIBRARY | Choose card from library search (0 = fail to find) |
| 20 | TOP_LIBRARY | Choose card to put on top of library |
| 21 | SHUFFLE | Choose whether to shuffle |
| 22 | PAYING_COSTS | Pay an optional cost |
| 23 | DIG_CHOICE | Choose creature/land from top N cards (e.g. Once Upon a Time) |
| 24 | SIDEBOARD_IN | Choose a card from sideboard to add to main deck (bo3) |
| 25 | SIDEBOARD_OUT | Choose a card from main deck to move to sideboard (bo3) |
| 26 | SIDEBOARD_DONE | Finish sideboarding (bo3) |

**Confirm slot convention:** mandatory attacker/blocker queries end with a confirm action. The Python env remaps `action = num_choices - 1` to `-1` before sending to the game.

### Observation space

Total: **33153 floats** (OBS_SIZE in `train/env.py`)

| Range | Size | Content |
|---|---|---|
| `[0:32555]` | 32555 | State vector (see `src/machine_io.h`) |
| `[32555:32619]` | 64 | Action categories, padded to MAX_ACTIONS (64), normalised by 26 |
| `[32619:32683]` | 64 | Action card IDs, padded to MAX_ACTIONS |
| `[32683:32747]` | 64 | Action controller_is_self flags, padded to MAX_ACTIONS |
| `[32747:32817]` | 70 | Hand cast costs (10 slots × 7 cost features) |
| `[32817:33153]` | 336 | Battlefield ability costs (48 slots × 7 cost features) |

State vector layout is documented in `src/machine_io.h`. Key indices: `obs[31]` = priority player is the active player (perspective-relative), `obs[32]` = priority player is Player A (absolute). To get `active_is_a`: `(obs[31] > 0.5) == (obs[32] > 0.5)`.

### Key files

- `train/env.py` — `RoboMageEnv` gymnasium wrapper; `ModelVsScriptedEnv` scripted-opponent wrapper; `SelfPlayEnv` self-play wrapper; `scripted_action` rule-based agent
- `train/extractor.py` — `CardGameExtractor` per-entity feature extractor for the policy network
- `train/train.py` — `MaskablePPO` training, baseline evaluation, observe mode, self-play
- `train/analysis.py` — post-game analysis tool (win rates, action frequencies, SHAP, replay from `.rmrec` recordings)
- `train/play.py` — interactive human-vs-model play
- `train/gen_card_costs.py` — regenerates `train/card_costs.py` from `src/card_vocab.h`
- `train/test_harness.py` — LLM test harness for card behavior verification (see Testing guidelines)
- `train/card_costs.py` — auto-generated cast-cost and ability-cost matrices (do not edit manually)
- `src/machine_io.h` — state vector layout documentation and constants
- `src/input_logger.cpp` — machine mode BQUERY emission, replay, and CLI input handling
- `src/card_vocab.h` — card name → vocab index mapping for one-hot encoding

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ceverettkoop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
