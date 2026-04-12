## gridworld

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A simple Reinforcement Learning demo implementing Q-Learning on a 5×5 grid world. The project is designed to run interactively in a Jupyter notebook with live HTML rendering of the grid.

## Running

The primary interface is `gridworld.ipynb`. Open and run it with:

```bash
jupyter notebook gridworld.ipynb
```

The notebook uses `%autoreload 2` so edits to `gridworld.py` are picked up automatically without restarting the kernel.

## Architecture

All logic lives in `gridworld.py`, which exports three classes and two functions used directly by the notebook:

- **`GridWorld`** — The environment. Manages a `size×size` grid with a start `(0,0)`, goal `(size-1, size-1)`, and hardcoded obstacles at `(2,2)` and `(1,3)`. State is stored internally as `(x, y)` tuples but exposed externally as flat integers (`state = x * size + y`). Actions are integers 0–3 mapping to down/up/left/right.

- **`QLearner`** — The agent. Maintains a Q-table `Q[n_states, n_actions]` initialized to zeros. Uses epsilon-greedy action selection with linear epsilon decay.

- **`train(env, agent, ...)`** — The training loop. Runs episodes, calls `agent.update()` after each step using the standard Q-learning Bellman update, and optionally renders the live grid via `env.display_grid()`.

- **`test_policy(env, agent, ...)`** — Greedy policy evaluation. Sets `epsilon=0` temporarily, records the path as `[(state, action)]` pairs, and renders the final learned path overlaid on the grid.

- **`GridWorld.display_grid()`** — Renders an HTML table into the Jupyter output area. Displays Q-values per cell, obstacles (☠️), goal (🍺), agent position (🚘), and path arrows. Uses `IPython.display` with a `display_id` handle for in-place updates.

## Dependencies

`numpy`, `IPython` (comes with Jupyter). No `requirements.txt` — install manually if needed:

```bash
pip install numpy jupyter
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/commanderfun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/commanderfun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
