## alphagozero-python-tensorflow

> Repo-specific notes for future Kilo sessions working on this folder.

# AGENTS.md — `new/` (PyTorch AGZ port)

Repo-specific notes for future Kilo sessions working on this folder.

## Scope

This is a **clean, modern, runnable** PyTorch reimplementation of the AGZ pipeline. The legacy TF 1.x code in the parent folder is **not** part of this; do not import from it. The interface contract is "self-play → replay buffer → train → evaluate against best", mirroring the original but without its many quirks.

## Stack

- **Python 3.10+**, **PyTorch 2.x**. No TF.
- `numpy` for Go state, `sgf` for any future SGF work.
- `requirements.txt` pins: `torch>=2.0`, `numpy`, `sgf>=0.5`. Add more deps with care.

## Layout

```
config.py        FLAGS + HPS (lazy: parses on first attribute access)
main.py          CLI dispatch — one function per mode
game/go.py       Position, rules, KO, captures, area scoring
game/features.py 17-plane encoder + 8 symmetries
model/network.py AGZNet — pre-activation ResNet + dual heads
model/resnet.py  PreActResidualBlock
mcts/mcts.py     PUCT MCTS, Dirichlet noise at root
training/
  dataset.py     ReplayBuffer (FIFO, fixed-size)
  selfplay.py    Self-play game generator
  train.py       Train step (policy + value loss + symmetry aug)
  evaluate.py    Head-to-head evaluator
tests/
  test_smoke.py  unittest suite (run with `python -m unittest new.tests.test_smoke`)
```

## Conventions

- **Config goes in `config.py`.** `FLAGS` and `HPS` are populated lazily via `__getattr__` so importing this module never consumes the test runner's argv. New hyperparameters go in `FLAGS` (argparse) or propagate via `HPS`.
- **No global state.** `Network` and `MCTS` are pure functions of their inputs. Pass `device` explicitly.
- **Symmetry augmentation lives in `game.features.apply_symmetry`.** Use `_mirror_policy` in `training/train.py` for the policy target — keep both in sync.
- **Replay buffer samples carry `_winner` as a side-channel attribute** (set by `attach_winner`). The value target is `+1` if `sample.player == sample._winner`, else `-1`.
- **Play index 0 is black.** `STONE_BLACK = 1`, `STONE_WHITE = 2`, `STONE_EMPTY = 0`. Pass is `(-1, -1)`.
- **MCTS root priors: Dirichlet noise on during self-play, off during evaluation.** Controlled by `dirichlet_alpha` + `dirichlet_epsilon` — passing both 0 disables noise.
- **Board size is configurable.** `model/network.py` accepts `board_size` and the heads use `n * n` flatten sizes. The CLI defaults to 19; the smoke tests use 9.
- **Import policy.** `training/__init__.py` only re-exports `ReplayBuffer` and `Sample` (from `dataset.py`). Other training modules must be imported explicitly (`from new.training.selfplay import ...`) to avoid chain-loading `config.py`.

## Quirks

- `Position._remove_group` uses **negative markers** (`-color`) so `_restore_group` can find the cells during undo. After capture is committed, cells are normalized back to `STONE_EMPTY` (0). Don't introduce a separate "removed" set — the marker is the cleanest signal.
- The MCTS scratch copy (`scratch = position.copy()`) is mutated during selection. This is the only place `Position.copy()` is hot; it's a numpy copy.
- `extract_features` walks `position.history` and replays it on a fresh `Position`. This is O(moves) per call — fine for batch sizes of 64, expensive for batches > 512. If you hit a wall here, cache `(features, position)` in the replay buffer.
- `apply_symmetry` and `_mirror_policy` must agree on how sym=0..7 maps to {rotations × reflections}. Look at both before changing either.
- `Position.score()` returns the area-score winner regardless of terminal state. `Position.winner()` requires `is_terminal()` to be True — use `score()` for truncated games (e.g. when self-play hits `max_moves`).
- `config.py` uses module-level `__getattr__` so `FLAGS` and `HPS` are only parsed on first access. If you ever add eager top-level code to `config.py` that touches `FLAGS`, you'll reintroduce the `python -m unittest` bug.

## Verifying changes

```bash
# Smoke tests (no GPU required).
python -m unittest new.tests.test_smoke

# Tiny end-to-end smoke through the CLI.
python new/main.py --mode=selfplay --n_games_per_iter=1 --n_simulations=5 --n_resid_blocks=1 --n_filters=16 --n_moves_per_game=10
python new/main.py --mode=train --n_train_steps=3 --batch_size=4 --n_resid_blocks=1 --n_filters=16
python new/main.py --mode=evaluate --n_eval_games=1 --n_simulations=5 --n_resid_blocks=1 --n_filters=16
```

## Tests

`new/tests/test_smoke.py` covers: Go rules (capture, alternation, marker cleanup), feature extraction (shape, symmetry round-trip), network (forward shape, no NaN), MCTS (visit policy sums to 1), replay buffer (add / sample / eviction). Run them with `python -m unittest new.tests.test_smoke`.

## Things future agents will get wrong

- Reading `STONE_BLACK = 1` and assuming `STONE_BLACK` is "true" / `1` in arithmetic. It's just an enum.
- Trying to use the legacy `model/` from the parent folder. Don't. This is its own clean implementation.
- Trying to "improve" `apply_symmetry` or `_mirror_policy` without re-deriving both. They are coupled.
- Adding TF 1.x code paths. This folder is PyTorch-only.
- Running the default config (`19` residual blocks, `800` simulations) on a CPU. Scale down first.
- Adding `from new.config import FLAGS` to `training/__init__.py`. That re-exports trigger chain-loading and break `python -m unittest`.

---
> Source: [yhyu13/AlphaGOZero-python-tensorflow](https://github.com/yhyu13/AlphaGOZero-python-tensorflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
