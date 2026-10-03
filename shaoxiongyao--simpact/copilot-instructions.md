## simpact

> `simpact` (all imports: `from simpact.x.y import Z`)

# simpact — development guide

## Package name
`simpact` (all imports: `from simpact.x.y import Z`)

## What this repo is
Offline real2sim (recorded RGB-D → simulatable scene) + a closed-loop VLM action
optimizer (propose → rollout → verify ↔ regress) for push / rope / dough / sweep.
Task specifics live in the registry (`simpact/tasks.py`); the single driver is
`scripts/optimize.py`; bundled trials + layout conventions are documented in
`examples/README.md`. Out of scope for this release: live capture, real-robot
execution, CEM, and measured shape gates for rope/dough (rope/dough success is
VLM-only; sweep has a measured coverage gate).

## Environment (uv)
Env is managed by uv. `uv sync --extra dev` builds `.venv` (Python 3.11.15,
pinned in `.python-version`); run commands with `uv run <cmd>`.
`pyproject.toml` + `uv.lock` are source of truth. The GPU stack targets an
RTX 5090 (Blackwell, sm_120): `torch==2.9.0+cu128` (cu128 index in
`pyproject.toml`) and `warp-lang==1.10.0` (1.11+ removed the `warp.torch`
submodule the MPM solver needs). Do not downgrade torch to a cu118 build — it
has no sm_120 kernels and crashes at runtime on the 5090.

## Environment gotcha
`import coacd` before `open3d` segfaults in this env — always import open3d
first. (Only `real2sim/convex_decomp.py` uses coacd; it handles the ordering.)

## Hardware guard rule
Always wrap frankx, pyrealsense2, warp imports in try/except ImportError.
Never import these at module top-level unconditionally.

## Paths & configuration
- Data/output roots via `simpact.utils.config` (`get_data_dir()` etc., .env-driven).
- Camera calibration is a keyed registry (`assets/calibration/<profile>/`)
  referenced **per scene** by `sim/scene.yaml`'s `camera: {profile}` — never a
  code default.
- Static rig data (calibration + the Franka gripper MuJoCo model) lives in
  top-level `assets/`, resolved via `real2sim/paths.get_assets_dir()`.
- Per-trial files resolve through `simpact/utils/layout.py::find_scene_file`
  (bundled `<trial>/{capture,sim,runs}` layout or flat external dirs).

## Do NOT modify (research-critical algorithm internals)
- simulators/mpm/solver.py internals (MPM warp kernels)
- simulators/arap/embed_deform_graph.py internals (ARAP deformation graph)
- prompts/**/*.txt prompt template CONTENTS (research prompts; filenames follow
  the task keys — push/rope/dough/sweep — and may be renamed)
- assets/robot/ (Franka gripper MuJoCo model data files)

## External perception repos (set via env, see .env.example)
- SIMPACT_GROUNDED_SAM2_DIR  (Grounded-SAM-2; repo clone OPTIONAL — the
  segmenter runs from the sam2 wheel; only the SAM2 checkpoint is required)
- SIMPACT_SAM2_CHECKPOINT    (sam2.1_hiera_large.pt)
- SIMPACT_FOUNDATIONPOSE_DIR (FoundationPose — 6D pose, push perception)
- SIMPACT_SAM3D_DIR          (SAM-3D — image-to-3D, push perception)

## Verification commands
```bash
uv run pytest tests/                          # full suite (CPU-safe)
git grep "AIzaSy"                             # must be empty (no baked API keys)
uv run python scripts/smoke_test_mpm.py       # GPU smoke (needs CUDA + warp)
uv run python scripts/smoke_test_arap.py      # ARAP energy decreases
bash scripts/reproduce_all.sh --quick         # end-to-end (auto-skips missing prereqs)
```

---
> Source: [ShaoxiongYao/simpact](https://github.com/ShaoxiongYao/simpact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
