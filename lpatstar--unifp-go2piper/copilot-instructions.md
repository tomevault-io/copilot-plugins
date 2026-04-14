## unifp-go2piper

> This repository is a Go2+Piper-oriented fork of UniFP for whole-body position-force control in Isaac Gym.

# AGENTS.md

## Project Identity

This repository is a Go2+Piper-oriented fork of UniFP for whole-body position-force control in Isaac Gym.

Primary task:
- `go2_piper_pos_force`

The repo is not a clean greenfield project. It keeps a large amount of upstream UniFP / B2+Z1 structure, and the Go2+Piper task is layered on top of that shared implementation.

## Environment Assumptions

- Python `3.8`
- Isaac Gym Preview 4
- Linux, typically Ubuntu `20.04` or `22.04`
- PyTorch and CUDA configured separately before running scripts

This repo is not meaningfully testable in a generic sandbox without a working Isaac Gym install and GPU runtime.

## Main Entry Points

Train:
```bash
cd legged_gym/scripts
python train_go2piperposforce.py --task=go2_piper_pos_force --headless
```

Play:
```bash
cd legged_gym/scripts
python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name>
```

Play with joint tracking draw export:
```bash
cd legged_gym/scripts
python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --draw
```

Keyboard teleop / manual inspection:
```bash
cd legged_gym/scripts
python keyplay_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name>
```

Automated evaluation:
```bash
cd legged_gym/scripts
python eval_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --headless
```

## Code Map

Go2-specific config:
- `legged_gym/envs/go2/go2_piper_pos_force_config.py`

Go2 task alias:
- `legged_gym/envs/go2/legged_robot_go2_piper_pos_force.py`

Shared environment implementation that actually contains most runtime logic:
- `legged_gym/envs/b2/legged_robot_b2z1_pos_force.py`

Shared base config inherited by Go2:
- `legged_gym/envs/b2/b2z1_pos_force_config.py`

Training stack:
- `legged_gym/b2_gym_learn/ppo_cse_pf/`

Task registration:
- `legged_gym/envs/__init__.py`
- `legged_gym/utils/task_registry.py`

## Architectural Notes

- `legged_robot_go2_piper_pos_force.py` is only a thin alias to the shared B2+Z1 environment class.
- Real Go2+Piper behavior is split between:
  - Go2 config overrides in `go2_piper_pos_force_config.py`
  - shared control / observation / viewer / force logic in `legged_robot_b2z1_pos_force.py`
- The Go2 config keeps the upstream control layout:
  - 12 leg joints plus the first 5 arm joints are policy-controlled
  - the final 3 Piper wrist / gripper joints are fixed-PD controlled
- Many changes that look "Go2-specific" still belong in the shared B2 environment because that is where command handling, force injection, debug drawing, viewer hotkeys, and rollout behavior live.

## Force And Command Conventions

- Commanded force and externally applied force are separate paths.
- Commanded EE force and commanded base force are stored in `commands`.
- External disturbance forces are applied through `self.forces`.
- In `play`, green arrows represent commanded force and blue arrows represent applied external force.
- For Go2+Piper, gripper force ranges are narrower than the inherited B2+Z1 defaults.

## Script Behavior Notes

- `train_go2piperposforce.py` delegates to the shared B2/Z1 training implementation and forces `args.task = "go2_piper_pos_force"` when not provided.
- `play_go2piperposforce.py` delegates to the shared play implementation and enables force visualization / play-side command-force behavior through module flags.
- `play_go2piperposforce.py --draw` runs a short rollout, saves joint command-vs-actual plots for one front-left leg and the task-relevant arm joints, then exits. The shared play script auto-resolves the correct arm joint names for Go2+Piper and B2+Z1.
- `keyplay_go2piperposforce.py` creates a single-env teleop setup, disables most randomization, and relies on viewer keyboard events for command updates.
- `eval_go2piperposforce.py` is the main reproducible checkpoint benchmark. Prefer it over ad hoc `play` sessions when comparing runs.
- `play_go2piperposforce.py`, `keyplay_go2piperposforce.py`, and `eval_go2piperposforce.py` all use checkpoint-resume loading. If `--load_run` and `--checkpoint` are omitted, the shared loader resolves to the latest run directory and then the latest saved `model_*.pt` checkpoint inside it.

## Play Draw Notes

- `play --draw` is the standard joint-level diagnostic path for tuning analysis.
- Default command:
  - `python play_go2piperposforce.py --task=go2_piper_pos_force --load_run=<run_name> --draw`
- Useful optional flag:
  - `--draw_steps <N>` to control how many play steps are recorded before the script exits
- Output directory:
  - `play_draws/`
- Expected files:
  - one leg joint tracking plot
  - one arm joint tracking plot
- During tuning, inspect the PNG plots directly and judge both:
  - whether actual tracks command
  - and whether command itself already shows obvious oscillation or other bad control patterns
- When a tuning sample does not already have draw plots, generate them before forming a tuning conclusion.

## Keyplay Notes

- `keyplay` intentionally disables random command resampling and random force events.
- Keyplay controls are split between:
  - `legged_gym/scripts/keyplay_go2piperposforce.py`
  - viewer-event handling inside `legged_gym/envs/b2/legged_robot_b2z1_pos_force.py`
- Numeric viewer hotkeys are disabled in key-command mode to avoid collisions with numpad EE controls.

## Evaluation Notes

- Automated evaluation writes reports under `eval_reports/`.
- Report folder names include the resolved run name and checkpoint so they can be matched back to tuning notes.
- `summary.json` is the machine-readable artifact and `summary.md` is the human-readable report.
- The report section named `Runtime Quality` summarizes runtime stability / posture / contact cleanliness / slip / smoothness.
- `Overall` is the weighted benchmark total, not the same quantity as `Runtime Quality`.

## Working Rules For Future Changes

- Preserve compatibility with the inherited B2+Z1 implementation. Any Go2+Piper change should avoid breaking the original B2+Z1 task, scripts, config expectations, or shared environment behavior unless an explicit compatibility break is requested.
- When changing Go2 behavior, first decide whether it belongs in:
  - Go2 config overrides
  - the shared B2 environment implementation
  - the script layer
- If changing keyboard control, force injection, viewer drawing, or command interpretation, inspect the shared B2 environment file before assuming the Go2 wrapper contains the logic.
- If changing evaluation metrics or report structure, keep `README.md` and `GO2_PIPER_EVAL_METRICS.md` aligned.
- Prefer targeted edits. This fork inherits duplicated concepts and upstream naming that are easy to break with broad refactors.

## Local Artifacts And Ignore Guidance

Do not commit these local or generated artifacts:
- `eval_reports/`
- `wandb/`
- `wandb_exports/`
- `logs/`
- `__pycache__/`
- local Isaac Gym install directories
- local paper PDFs or extracted text notes

## Validation Expectations

- For code-only changes, validate by reading the affected config, script, and shared environment paths together.
- For behavior changes, prefer running the narrowest relevant entry point:
  - `play` for quick viewer inspection
  - `keyplay` for manual command-path debugging
  - `eval` for reproducible checkpoint comparison
- Do not first attempt Isaac Gym runtime commands in the default sandbox when the task clearly needs real execution. For `play`, `keyplay`, `eval`, or other commands that rely on GPU access, Isaac Gym native bindings, viewer access, or writable PyTorch extension caches such as `~/.cache/torch_extensions`, request escalated execution immediately instead of failing once in the sandbox and retrying afterward.
- If Isaac Gym is unavailable, state that runtime validation could not be executed rather than guessing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LPatstar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
