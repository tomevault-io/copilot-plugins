## battleship-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Battleship game with a CNN-based AI opponent. Players compete against an AI that learns from gameplay using a Convolutional Neural Network trained with REINFORCE policy gradient, with a heuristic fallback strategy. Built with Python, PyTorch, and Tkinter.

## Running the Project

```bash
# GUI mode (default)
python root.py

# Headless training mode
python root.py --no-gui --games 500

# GUI directly
python visual_root.py

# Test CNN model structure
python enhanced_cnn.py
```

## Dependencies

No requirements.txt exists. Install manually:
```bash
pip install torch numpy pyyaml
```
TensorBoard is optional (for training visualization): `pip install tensorboard`

## Architecture

**Entry points:** `root.py` (main, dispatches to GUI or training) and `visual_root.py` (GUI-only).

**Core modules:**
- `basic_structure.py` — Game engine: `GameBoard`, `Ship`, `Game` classes. Handles board state, ship placement (`auto_place_ships` with no-touch validation), shot processing, and state tensor generation (`get_state_tensor` produces a 4-channel fog-of-war tensor).
- `enhanced_cnn.py` — `EnhancedCNN`: 3 conv layers (with batch norm, no pooling) preserving 10x10 spatial resolution + 1x1 conv output layer. Single flat output of 100 logits (one per cell). No epsilon-greedy — uses softmax sampling.
- `model_op.py` — REINFORCE policy gradient training (`train_model`, `start_training`), inference (`get_ai_prediction` with `-inf` masking), model save/load (`model_checkpoint.pth`). Logs to TensorBoard.
- `config.py` — Global switches (`ENABLE_GUI`, `USE_AI_MODEL`, `ENABLE_LOGGING`), `DEFAULT_CONFIG` dict, YAML config loading, logging setup (file + colored console output), `CellState` and `ShipType` enums.

**Visual package (`Visual/`):**
- `gui.py` — `BattleshipGUI`: Tkinter dual-board UI with strategy toggle (Neural Network vs Heuristic), AI vs AI simulation mode, probability overlay. AI gets extra turn on hit (standard Battleship rules).
- `ai_interface.py` — `load_ai_model()` and `get_ai_prediction()` (softmax sampling with `-inf` masking) / `get_heuristic_prediction()` bridge between GUI and model.
- `renderer.py` — `BoardRenderer.draw_board()`: renders 10x10 grid with color-coded cells and optional probability heatmap.

## Key Design Decisions

- **Single flat output:** The CNN outputs 100 logits (one per board cell) instead of dual X/Y heads. This allows the model to learn correlations between specific cell positions.
- **No pooling architecture:** All conv layers use `padding=1` with no pooling, preserving the 10x10 spatial resolution throughout. This is critical for a 10x10 board — pooling destroys spatial information.
- **Fog of war state tensor:** 4 channels — (1) active hits (wounded, not sunk), (2) sunk ships, (3) misses, (4) probability heatmap. The AI never sees ship positions — only results of its own shots.
- **REINFORCE with baseline:** Batch update at end of each game with normalized rewards. Entropy bonus (coeff=0.05) from full softmax distribution prevents over-confidence.
- **Temperature sampling:** Training uses temperature=1.5 for softmax sampling to maintain exploration. Inference uses standard softmax.
- **Reward shaping:** Sunk ship = 3.0, hit adjacent to existing hit = 2.0 (encourages finishing wounded ships), new hit = 1.0, miss = -0.1.
- **`-inf` masking:** Already-shot cells get logit = `-inf` before softmax, making them impossible to select. Safety fallback to heuristic if masking fails.
- **Heuristic fallback:** `generate_gradient_probabilities()` creates a probability heatmap: boosts cardinal directions around active hits (ships are linear), zeroes out sunk ship areas + neighbors (ships can't touch), penalizes miss areas.
- **Ship placement validation:** Ships cannot overlap or touch (even diagonally). `place_ship()` checks that all target cells are `UNKNOWN` and no adjacent cells contain ships.
- **Extra turn on hit:** Both in manual play and simulation, a player gets another turn after hitting a ship (standard Battleship rules).
- **Ship fleet:** 10 ships total — 1 battleship (4 cells), 2 cruisers (3 cells), 3 destroyers (2 cells), 4 submarines (1 cell).

## Training Performance

- Random baseline: ~20% hit rate
- After ~3000 epochs: 35-40% hit rate (stable), peaks up to 52%
- Best game observed: 38 moves to find all 20 ship cells (52.6% hit rate)
- Model effectively learns to finish wounded ships and avoid sunk ship areas

## Configuration

Config loads from `config.yaml` (auto-created with defaults if missing). Key defaults in `config.py`:
- Learning rate: 0.001, optimizer: Adam, epochs: 500, checkpoint interval: 10
- Model saved to `model_checkpoint.pth`
- Logs: `battleship_game.log` (game events), `training.log` (training metrics)

## Language

Code and comments are in English. Project documentation (`About.txt`) is in Russian.

---
> Source: [VoiceOfVoidSF/battleship-ai](https://github.com/VoiceOfVoidSF/battleship-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
