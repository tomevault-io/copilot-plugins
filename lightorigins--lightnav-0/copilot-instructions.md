## lightnav-0

> Working notes for coding agents. The goal this file optimises for: **bring up

# AGENTS.md

Working notes for coding agents. The goal this file optimises for: **bring up
`lightnav-serve` and drive the MuJoCo demo against it, end to end, without a robot.**
Human-facing documentation lives in [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md).

## Repository shape

```text
src/lightnav/
  inference/   model backends (hf, vllm_local), the vLLM 0.19.x patch, RVQ decoding
  serving/     lightnav-serve: WebSocket server, session state, micro-batch scheduler
  cli/         predict, ws_client, eval_habitat, eval_merge, render entry points
  habitat/     Habitat evaluation client (talks to habitat_server over ZMQ)
  viz/         trajectory / pointing overlay rendering
habitat_server/  Habitat env server — separate Python 3.9 conda env, not this venv
evt_bench/       EVT-Bench client agent + a patch applied to a third-party checkout
robot_deploy/    ROS 2 on-robot stack (real hardware)
mujoco_demo/     standalone MuJoCo demo — its own uv project, no ROS, no GPU needed
scripts/         eval_habitat.sh, eval_evt_bench.sh, start_servers.sh, smoke_gpu.sh
tests/           CPU-only, fake engines and synthetic tokens
```

## Commands

```bash
pip install -e ".[test]"    # test extra is not part of the runtime extras
make check                  # ruff check .
make test                   # pytest -m "not gpu"  — CPU only, no checkpoint needed
MODEL_PATH=... bash scripts/smoke_gpu.sh   # every real-hardware path, minimal scale
```

`make test` is the gate that needs no GPU and no weights: run it after any change to
`src/`. `scripts/smoke_gpu.sh` prints a PASS / FAIL / SKIP table and is the gate that
needs a GPU and a checkpoint.

## Bring-up: server + MuJoCo demo

Two processes. The server needs a CUDA GPU; the demo does not and may run on another
machine.

### 1. Environment

```bash
python3.11 -m venv .venv && source .venv/bin/activate
pip install -e ".[vllm,video]"
```

Check before going further: `python -c "import torch; print(torch.cuda.is_available())"`.
If it prints `False`, `--backend vllm_local` will not start — stop and report, do not
fall back silently.

### 2. Checkpoint — needs a human

```bash
hf download LightOriginsHQ/LightNav-0 --local-dir checkpoints/LightNav-0
```

Requires a Hugging Face login (`hf auth login`) and, if the repo is gated, an approved
access request. An agent cannot complete either. If `hf download` fails on auth, stop and
ask; do not substitute another checkpoint.

A valid checkpoint directory holds `config.json`, `model*.safetensors`, `tokenizer*`,
`processor_config.json`, `eval_config.json` and an `action_tokenizer/` bundle. When
`action_tokenizer/` is absent the server needs `--action_tokenizer_bundle <path> --horizon 10`
passed explicitly; a released checkpoint does not.

### 3. Server

```bash
PORT=8050 CUDA_VISIBLE_DEVICES=0 lightnav-serve \
    --task vln --model_path checkpoints/LightNav-0 --backend vllm_local
```

Use `--task vln` for the demo — it selects the navigation prompt. `--task tracking`
selects the follow-a-target prompt and is the wrong one here.

The server runs a synthetic warm-up inference *before* binding the port, so the port
opening is a real readiness signal, not a race. Poll for it rather than sleeping, or pass
`--ready_file <path>` and wait for that file to appear. First start is slow: the engine
loads weights and warms up.

### 4. Demo

```bash
cd mujoco_demo && ./run.sh --vln-server ws://127.0.0.1:8050
```

Needs [uv](https://docs.astral.sh/uv/) and Python 3.11+; it resolves its own dependencies
and does not use the venv from step 1. Then open <http://127.0.0.1:8088>, type an
instruction, press **Start VLN**. Use `--host 0.0.0.0` when the browser is on another
machine, and `ws://<gpu-host>:8050` when the server is.

### 5. Verify

The run is working when the console shows a non-zero **LATENCY**, **WAYPOINTS** counting
up, and the blue waypoint trail moving ahead of the robot. `RUNNING · MPC tracking the VLN
path` in the status line means the client loop and the controller are both live.

Without a reachable server the page still loads and **Take control** drives with WASD — so
a page that renders proves nothing about the server. Check the telemetry, not the page.

## Constraints that bite

- **vLLM is pinned to 0.19.1 exactly.** `inference/vllm_utils.py` binds private APIs
  (`BaseRenderer._process_multimodal`, `Qwen3VL.get_mrope_input_positions`); a runtime
  guard fails loudly on any other version. `LIGHTNAV_SKIP_VERSION_GUARD=1` bypasses it and
  is unsupported — do not set it to make an install "work".
- **`nvidia-cutlass-dsl` is pinned in the `vllm` extra** and must stay pinned. Letting pip
  resolve it freely picks a dev build without `cutlass.cute.core.ThrMma`, and the engine
  dies during startup.
- **Blackwell sm_103 (B300 / B30Z).** The stock cu12.8 torch wheel cannot JIT for it and
  the `hf` backend dies in the Qwen3-VL vision tower with
  `nvrtc: error: invalid value for --gpu-architecture`. Install
  `torch==2.10.0+cu129` / `torchvision==0.25.0+cu129` from the cu129 index — the `+cu129`
  suffix is required, since pip treats a bare `torch==2.10.0` as already satisfied.
  `vllm_local` is unaffected.
- **Frames** are HWC `uint8` RGB at any resolution; they are resized to the checkpoint's
  `video_size` internally. Pointing pixels come back in the *client's* frame size.
- **Waypoints carry no time base.** A row is `[forward_m, lateral_m (+ = left),
  yaw_rad (+ = ccw)]`; command `v = forward / dt`, `w = yaw / dt` with your control period.
- **Habitat is a separate Python 3.9 conda env**, never this venv. Installing
  `habitat-lab` into the inference venv will break it.

## Out of scope for an agent

- **Habitat evaluation.** MP3D needs a signed agreement, HM3D needs approved Hugging Face
  access, and the env is a separate conda install. Prepare commands, but the downloads
  need a human.
- **EVT-Bench.** Requires cloning a third-party repository and applying
  `evt_bench/run_py.patch` to it. Ask before touching anything outside this repo.
- **Real robots** (`robot_deploy/`). Physical hardware, network setup and safety. Never
  start a robot-side launch file as part of an automated task.

## Conventions

- Match the surrounding code; `make check` (ruff) is authoritative on style.
- New CPU-testable behaviour gets a test under `tests/`. Mark anything needing CUDA or
  weights with `gpu` so `make test` keeps skipping it.
- Every server / CLI flag is documented in [docs/CONFIGURATION.md](docs/CONFIGURATION.md);
  add new ones there in the same pass.

---
> Source: [lightorigins/LightNav-0](https://github.com/lightorigins/LightNav-0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
