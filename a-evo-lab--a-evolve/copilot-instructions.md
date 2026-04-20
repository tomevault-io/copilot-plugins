## a-evolve

> **a-evolve** is the universal infrastructure for evolving AI agents through self-improvement. It enables automatic, data-driven optimization of agents across any domain using any evolution algorithm.

# CLAUDE.md

## Project Overview

**a-evolve** is the universal infrastructure for evolving AI agents through self-improvement. It enables automatic, data-driven optimization of agents across any domain using any evolution algorithm.

This branch (`arc-agi-3-dev`) focuses on ARC-AGI-3 -- an interactive game benchmark from ARC Prize that measures reasoning through game-playing efficiency (RHAE score).

## Repository Structure

```
agent_evolve/
  agents/
    arc/                    # ARC-AGI-3 agents
      basic_agent.py        # Minimal: text obs, 1 LLM call per action, boto3 direct
      agent.py              # Code-driven game loop + per-action Bedrock calls
      strands_agent.py      # Strands SDK with tools (observe/action/code_exec)
      game_loop.py          # Core while loop: choose_action -> env.step -> repeat
      frame.py              # Frame class with diff/render/find/color_counts/bounding_box
      colors.py             # 16-color palette (names, hex, RGBA)
      grid_render.py        # Grid-to-PNG for multimodal input
    swe/                    # SWE-bench agent
    terminal/               # Terminal-Bench agent
    clawcode/               # Claw-code agent
  benchmarks/
    arc_agi3/               # ARC-AGI-3 benchmark adapter
      benchmark.py          # get_tasks() from arcade, evaluate() with RHAE scoring
    base.py                 # BenchmarkAdapter ABC
  protocol/
    base_agent.py           # BaseAgent ABC: solve(task) -> Trajectory
  api.py                    # Evolver top-level API + registries
  types.py                  # Task, Trajectory, Feedback, Observation, SkillMeta
seed_workspaces/
  arc/                      # Evolvable workspace for ARC agent
    manifest.yaml           # entrypoint + evolvable_layers
    prompts/system.md       # System prompt (evolved by a-evolve)
    skills/                 # Learned skills (evolved)
    memory/                 # Episodic memory (evolved)
examples/
  arc_examples/
    play_ls20.py            # Play LS20 with real-time display on port 7889
    eval_2games.py          # Evaluate on 2 games
    evolve_arc.py           # Full evolution loop
    serve_replay.py         # Serve replay visualization
```

## ARC-AGI-3 Key Concepts

### Game Environment (arc-agi SDK)
- `pip install arc-agi` (requires Python 3.12+)
- `arcade = arc_agi.Arcade()` -> `env = arcade.make(game_id)`
- `raw = env.reset()` -> `raw = env.step(GameAction.ACTION1)`
- `FrameDataRaw`: frame (64x64 numpy), state, levels_completed, win_levels, available_actions
- `env.environment_info.baseline_actions` = human action count per level

### Available Actions (per game)
- ACTION1-4: directional (up/down/left/right in keyboard games)
- ACTION5: contextual interact
- ACTION6: click at (x,y) coordinates 0-63
- ACTION7: undo
- RESET: restart level
- Not all actions available in every game -- check `available_actions`

### 25 Games, 181 Levels Total
- Tags: keyboard (4), click (7), keyboard_click (13)
- Human baselines: 15-550 actions per level
- Scoring: RHAE = (human_actions / agent_actions)^2, averaged across levels and games

### Frame Helpers (frame.py)
```python
frame = Frame(grid)
frame.render(y_ticks=True, x_ticks=True, crop=(x1,y1,x2,y2))
frame.diff(other_frame) -> list[DiffRegion]
frame.change_summary(other_frame) -> "4 cells changed across 1 region..."
frame.find(*colors) -> [(x, y, value), ...]
frame.color_counts() -> {color_int: count}
frame.bounding_box(*colors) -> (x1, y1, x2, y2)
```

## Development

### Setup
```bash
uv venv --python 3.12 .venv
uv pip install -e . arc-agi strands-agents strands-agents-bedrock boto3 numpy flask
```

### Run LS20 with real-time display
```bash
.venv/bin/python examples/arc_examples/play_ls20.py
# Open http://localhost:7889 in browser
```

### Run evaluation
```bash
.venv/bin/python examples/arc_examples/eval_2games.py
```

### Agent Architecture Pattern
```python
# Code-driven game loop (not LLM-driven)
arcade = Arcade()
env = arcade.make(game_id)
raw = env.reset()

while action_count < max_actions:
    observation = format_observation(frame, meta)
    action = llm_call(observation)        # ONE call per step
    raw = env.step(GameAction.from_name(action))
    frame = convert_frame_data(raw)
    action_count += 1
```

### Key Insight: Perception Matters
- Text-only (64x64 grid as characters): LLM spends 80% of calls analyzing, ~1 action per 10 calls
- Vision (PNG image): LLM acts immediately, ~1 action per call
- Vision + batched code: multiple actions per call via execute_actions(code)
- Best: vision input + diff images showing what changed

### Prompt Design Principle
> "Every time something changes that you did NOT expect from your action, you have discovered new information about the game mechanics."

Unexpected changes are the most valuable signals. When something surprises the agent, it should STOP and reason about what rule could explain it.

## Branch History

- `main`: upstream a-evolve with SWE-bench, MCP-Atlas, Terminal-Bench, SkillBench
- `arc-agi-3`: ARC-AGI-3 benchmark + agent + seed workspace
- `arc-agi-3-dev`: active development -- basic agent, vision experiments, LS20 testing

## Test Results

### LS20 Level 1 (with explicit instructions)
- **Completed in 19 actions** (human baseline: 21)
- Agent followed Phase 1 (move to white cross) + Phase 2 (exit through corridor) correctly
- Level 2+ without instructions: not completed in remaining 81 actions

### Pure LLM (no instructions)
- Text-only: 0/7 levels in 300 actions (agent couldn't parse 64x64 grid)
- Vision: 0/7 levels in 300 actions (better perception but no game understanding)
- Vision + batched: 0/7 levels (faster but same reasoning gap)

### SOTA Context
- StochasticGoose (1st): 12.58% RHAE (CNN + RL)
- Blind Squirrel (2nd): 6.71% (State graph + ResNet18)
- Pure LLM approaches: ~0% (consistent across all attempts)

---
> Source: [A-EVO-Lab/a-evolve](https://github.com/A-EVO-Lab/a-evolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
