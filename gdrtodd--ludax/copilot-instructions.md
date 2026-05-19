## ludax

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ludax is a JAX-based board game framework with a DSL compiler. It transforms `.ldx` game description files into hardware-accelerated JAX environments compatible with RL training pipelines.


## Setup and Commands

**Install (development):**
```bash
pip install -e .[gui]
```

**Run a basic example:**
```bash
python examples/01-basic-usage/play_one.py
```

**Launch the interactive GUI:**
```bash
python examples/02-ludax_gui/interactive.py
```

**Run AlphaZero training:**
```bash
python examples/03-alpha-zero/train.py
```

There is no test suite — examples serve as integration tests.

## Architecture

The core pipeline transforms `.ldx` files into JAX functions:

```
.ldx game file
    → lark parser (grammar.lark)
    → GameInfoExtractor (game_info.py)   → GameInfo + GameState namedtuple
    → GameRuleParser (game_parser.py)    → compiled JAX functions
    → LudaxEnvironment (environment.py)  → gym-like step/init/observe API
```

### Key Components

**`config.py`** — Central types and constants. All dtype globals live here:
- `BOARD_DTYPE = jnp.int8` (board cells, piece/player indices)
- `ACTION_DTYPE = jnp.int16` (action indices, step counters)
- `REWARD_DTYPE = jnp.float32` (rewards, value estimates)
- Player constants: `EMPTY = -1`, `P1 = 0`, `P2 = 1`
- `State` dataclass (JAX pytree-compatible)

**`game_parser.py`** (~111KB) — The core compiler. A Lark `Transformer` that compiles game rules into JAX functions: `start_rules`, `apply_action_fn`, `legal_action_mask_fn`, `apply_effects_fn`, `next_phase_fn`, `next_player_fn`, `end_rules`. Uses precomputed adjacency/slide lookup tables for performance.

**`game_info.py`** — A Lark `Visitor` (`GameInfoExtractor`) that extracts static metadata (board shape, piece types, observation shape) and dynamically builds a `GameState` namedtuple with only the attributes needed for that specific game.

**`environment.py`** — The user-facing API: `LudaxEnvironment(game_path=...)`. Main methods: `init(rng)`, `step(state, action)`, `observe(state, player_id)`. Hard max of 2000 steps before truncation.

**`utils.py`** — Board geometry, adjacency/slide lookup table construction, connected-components computation for Go-like rules.

**`struct.py`** — Flax-derived frozen dataclass decorator for JAX pytree registration. Required for all dataclasses passed through `jax.jit`/`jax.vmap`.

**`policies/`** — Pluggable agents: `mcts.py` (vanilla UCT), `mctx.py` (DeepMind mctx wrapper), `negamax.py` (alpha-beta), `beam.py`, `value.py` (heuristics), `simple.py` (random/greedy).

**`gui/`** — Flask web app with SVG board rendering. `render.py` handles square, rectangular, and hexagonal boards.

### Game DSL (.ldx format)

Games live in `src/ludax/games/` (bundled). Example:
```
(game "Tic-Tac-Toe"
    (players 2)
    (equipment (board (square 3)))
    (rules
        (play (repeat (P1 P2) (place (destination empty))))
        (end
            (if (line 3) (mover win))
            (if (full_board) (draw))
        )
    )
)
```

The grammar is defined in `src/ludax/grammar.lark`. Move types: `slide`, `hop`, `step` (piece movement) and `place` (placement). Effects: `capture`, `promote`, `extra_turn`, `flip`. End conditions: `line N`, `connected`, `captured_all`, `by_score`.

### JAX Conventions

- All state passed through `jax.jit`/`jax.vmap` must be a pytree — use `@struct.dataclass` (from `struct.py`) for new dataclasses.
- Use dtype globals from `config.py` when creating new arrays; do not hardcode `jnp.int8`/`jnp.int16`/`jnp.float32`.
- `jnp.bool_` is kept hardcoded (not quantization-relevant).
- The GUI's `routes.py` has one `jnp.int32` for label computation intentionally left as-is.

### HPC / Container Environment

Development runs on HPC with Apptainer containers (`jax.toml`). Base image: `nvcr.io/nvidia/jax:26.01-py3`. The `interactive.sh` script launches a zellij session via `srun`.

---
> Source: [gdrtodd/ludax](https://github.com/gdrtodd/ludax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
