## arm-farm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Everything runs through `uv` (Python 3.12). The `sim` extra pulls mjlab,
mujoco-warp, viser, onnxruntime — only sync it when touching the sim
path, since it's heavy (cu128 torch on Linux).

```bash
uv sync                  # base: lerobot + feetech + core_scripts
uv sync --extra sim      # adds mjlab + mujoco-warp + onnxruntime
uv sync --extra training # adds accelerate + wandb (per-policy: act, diffusion, smolvla, pi)
```

Lint, format, type-check:

```bash
uv run ruff check
uv run ruff format
uv run ty check
```

Tests (sim suite — needs the `sim` extra; tests `importorskip` mjlab/mjviser
so a base sync no-ops):

```bash
uv run --extra sim python -m pytest src/arm_farm/sim/tests/ -q
uv run --extra sim python -m pytest src/arm_farm/sim/tests/test_preview.py::test_compute_actor_obs_matches_cube_task_layout -q
```

`pyproject.toml` declares `[project.scripts]` `preview` and `convert2onnx`
which become console entries inside the venv:

```bash
uv run --extra sim preview                                 # native MuJoCo + viser preview (Mac-fast)
uv run --extra sim preview --task=Cube --action=sine       # scripted ctrl
uv run --extra sim preview --wandb-run-path entity/proj/id # wandb dropdown
uv run --extra sim convert2onnx <entity>/<proj>/<run_id>   # .pt → .onnx in a wandb run
```

Project-level shell wrappers under `scripts/` and `scripts/sim/` set
`HF_LEROBOT_HOME=<repo>/data` and forward extras to lerobot/mjlab CLIs;
prefer them over invoking lerobot/mjlab CLIs directly.

## Architecture

The repo has two halves built on top of lerobot:

1. **Hardware path** (real SO-ARM101 arms) — thin scripts around lerobot's
   record/teleoperate/calibrate/train CLIs. `src/arm_farm/hardware.py` holds
   typed lerobot config presets driven by `.env` vars; the shell scripts in
   `scripts/` pass those same env vars on the CLI. `scripts/train.sh
   <act|diffusion|smolvla|pi> <dataset>` switches `--extra` so heavy policy
   deps only land when used.

2. **Sim path** (`src/arm_farm/sim/`) — registers four mjlab tasks and a
   custom lerobot Robot, plus a Mac-fast preview tool.

### Sim package wiring

`pyproject.toml` declares `[project.entry-points."mjlab.tasks"] arm_farm =
"arm_farm.sim"`. mjlab imports that on startup, which runs
`arm_farm/sim/__init__.py`, which imports `lerobot_robot` and `tasks` — both
register on import:

- `tasks/{cube,cube_rgb,cube_depth,play}/__init__.py` each call
  `register_mjlab_task(...)` with `runner_cls=ArmFarmManipulationRunner`
  (the local `runner.py` subclass — adds per-checkpoint ONNX naming and
  verbose export errors over mjlab's `ManipulationOnPolicyRunner`).
- `lerobot_robot/__init__.py` registers `MujocoSO101RobotConfig` as a
  draccus subclass, exposing `--robot.type=mujoco_so101` to the lerobot CLI
  for dataset replay through an mjlab env.

`tasks/lift_cube_env_cfg.py` is the shared factory for the three lift-cube
variants — `Cube` is state-only, `Cube-Rgb` and `Cube-Depth` use
`add_camera_obs(...)` to swap the actor's privileged `ee_to_cube` /
`cube_to_goal` for a plain `goal_position` so the policy must learn from
pixels (the critic keeps the privileged state). Per-task obs/action shapes
are tabulated in `src/arm_farm/sim/tasks/README.md`.

Defaults to know: 200 Hz physics, decimation=7 → ~28.6 Hz policy/camera obs
(matches the 30 Hz lerobot dataset rate). `play=True` mode forces
`num_envs=4`, infinite episode, no obs corruption, no curriculum;
training uses `num_envs=4096`.

### preview.py (Mac-fast viewer)

Bypasses mjwarp entirely — reuses mjlab's `Scene` for spec composition but
drives rollouts under native `mujoco.mj_step`, ~10x faster on Apple silicon
than mjlab's manager-based env. To stay term-for-term compatible with
trained policies it reimplements:

- Actor obs (`_compute_actor_obs`): reads the ONNX `observation_names`
  metadata and assembles `joint_pos`/`joint_vel`/`ee_to_cube`/`cube_to_goal`/
  `goal_position`/`actions` straight off `mj_data` — anything else raises
  `NotImplementedError`. Add new terms here when extending env_cfg.
- Goal sampler (`_CubeSampler`): native clone of mjlab's `LiftingCommand`,
  reading ranges from the loaded `env_cfg`. Resamples on reset and every
  `command_resample_period_s`.
- Reward terms (`_build_reward_ctx`): each env_cfg reward func has a
  hand-written native equivalent for the live-plot panel. The
  `test_reward_ctx_covers_every_env_cfg_term` test fails when a new reward
  term is added without a native equivalent.

Threading model: viewer thread runs `mj_step` + writes obs frames to a
slot; a separate panel-pump thread does GL renders + PNG-encodes for the
GUI panels (camera previews, reward plots) so the 60 Hz hot path stays
clean. `mujoco.Renderer` is GL-context-bound to its constructing thread —
preview renderers are built lazily inside the panel-pump thread on first
use. Don't move renderer construction onto the viewer thread.

ONNX checkpoints come from mjlab's runner with metadata embedded
(`joint_names`, `observation_names`, `action_scale`, `default_joint_pos`).
The wandb dropdown lists `.onnx` + `.pt`; `.pt`-only runs need
`convert2onnx` first (which builds the env once, ~30 s mjwarp warmup,
then converts each `.pt`).

### Storage

`HF_LEROBOT_HOME=<repo>/data` is set by every script. `data/`, `outputs/`,
`wandb/` are gitignored. `src/arm_farm/sim/reference/` currently holds
local read-only checkouts of upstream repos (mjlab, lerobot-sim2real,
…) for browsing — untracked, will be removed in a future commit, never
import from it.

### .env loading

Scripts source `.env` themselves; Python entry points (`preview`,
`convert2onnx`) call `arm_farm.sim._env.load_env()` at startup, which
parses `<repo>/.env` and `setdefault`s `HF_LEROBOT_HOME=<repo>/data`. For
ad-hoc Python: `set -a; source .env; set +a`.

## Conventions

- **Long lines (`E501`) are explicitly disabled** — let `ruff format` wrap
  them. Don't reflow by hand to fit 88/100/110.
- **`ty` C-extension noise**: `pyproject.toml` has overrides silencing
  `unresolved-attribute` / `invalid-argument-type` / `not-subscriptable`
  for files heavy in `mujoco.MjModel` direct attribute access (preview.py,
  so101_constants.py, the sweep harnesses). Don't sprinkle `# ty: ignore`
  in those files; add them to the override list instead.
- **Vendored MJCF**: `src/arm_farm/sim/assets/so101/xmls/` is from
  TheRobotStudio/SO-ARM100 (Apache-2.0). Don't edit in place — modify the
  spec programmatically in `assets/so101/so101_constants.py:get_spec()`.

---
> Source: [KyleM73/arm_farm](https://github.com/KyleM73/arm_farm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
