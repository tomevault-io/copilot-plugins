## projpiece

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProjPiece is an LLM-driven JRPG battle system that uses Google GenAI (Gemini) for combat resolution. Game state is represented as XML, and the LLM modifies state via structured patch operations.

## Commands

```bash
# Install dependencies (using uv)
uv sync

# Run the game with built-in demo board
uv run python -m projpiece.game

# Run with a custom board file
uv run python -m projpiece.game --board boards/demo.xml
uv run python -m projpiece.game -b boards/volcano_boss.xml

# Run individual module tests (modules have __main__ blocks)
uv run python -m projpiece.action_tree
uv run python -m projpiece.sigil
uv run python -m projpiece.xml_patch
uv run python -m projpiece.hud
```

## Environment Setup

Set `GOOGLE_API_KEY` or `GEMINI_API_KEY` environment variable (uses Gemini Flash models).

## Architecture

### Game Loop Flow (game.py)
1. **Round Start** - Process `round-start` sigils (auto-damage, status ticks)
2. **Planning** - LLM generates action trees for party, pre-plans enemy actions
3. **Selection** - Player navigates action trees via keyboard UI
4. **Resolution** - LLM resolves each action in turn order via XML patches
5. **Round End** - Process `round-end` sigils (decrement counters, remove expired)
6. **Cleanup** - LLM removes dead entities, verifies state consistency

### Key Components

**Action Tree DSL (action_tree.py)** - Custom format for representing hierarchical action menus:
- `$ref = val1, val2` defines reusable leaf references
- `@subtree:` defines reusable branches shared across characters
- Parsed into `ActionTree` with `ActionNode` children

**Sigil System (sigil.py)** - Declarative hooks embedded in XML for game logic:
- `auto`: Deterministic state changes executed locally (no LLM)
- `hint`: Context injected into LLM prompts
- `task`: LLM-executed tasks (narration, complex effects)
- Phases: `round-start`, `round-end`, `planning`, `resolution`, `actor-turn-start`, `actor-turn-end`

**XML Patch Engine (xml_patch.py)** - Operations for state modification:
- `set_attr`: Modify attribute with `{{ @attr - N | min: 0 }}` expression syntax
- `insert`/`remove`/`move`: DOM manipulation
- Expressions use double braces `{{ }}` with filters (`min:`, `max:`, `clamp:`)
- References: `@attr` (attribute), `@owner` (parent entity id), `$xpath`

**Formula Evaluation (formula.py)** - Lua 5.4 sandboxed execution:
- Helpers: `dice()`, `chance()`, `clamp()`
- Entity access via `self`, `target`, `state`, or direct id globals (e.g., `hero.hp`, `goblin1.atk`)

**HUD (hud.py)** - Terminal UI using Rich:
- Data-driven from `<ui-hints>` in XML
- Keyboard navigation: arrows to move, z/Enter to select, x/Esc to go back, d/Tab for debug view

### LLM Integration Pattern

The game uses Google GenAI function calling. Key tools declared in game.py:
- `generate_actions`: Outputs action tree DSL for party
- `generate_enemy_actions`: Pre-plans enemy turn actions
- `apply_patch`: Executes XML patch operations with narration
- `eval_formula`: Runs Lua snippets for dice/calculations
- `qed`: Signals action resolution is complete

### Board Files

XML files in `boards/` define game scenarios. Structure:
- `<ui-hints>`: Display configuration
- `<party>`: Characters with skills, inventory
- `<battlefield>`: Enemies, turn order
- `<sigil>` elements throughout for game logic hooks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RuneBlaze)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RuneBlaze)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
