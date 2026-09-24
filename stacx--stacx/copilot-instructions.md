## stacx

> handles, then return immediately. The driver stream goes to a log that

# STACX Agent Runbook

This is the first file a coding agent should read in this repo. It is a router
and a set of verified happy paths. Prefer these commands over scattered ad hoc docs.

Rules for agents:

- Work from the repo root unless a command says otherwise.
- Read only the section needed for the task, plus its referenced detail doc.
- Before launching any shell script, read its header comments. The headers are
  part of the run documentation and list current knobs/defaults.
- Do not start a full multi-hour run until the matching smoke or data/setup
  verification passes.
- Large models, task trees, image builds, logs, results, and checkpoints live
  outside git under `external/`, `/scratch/$USER/...`, or a configured profile
  scratch directory.

## 0. Big Picture

STACX trains and evaluates coding agents in real Docker sandboxes. Agents run
in one of **two lanes**, and every task in this runbook is one cell of the
lane × train/eval matrix:

- **In-house agent lane** — the agent loop is implemented in this repo
  (`rl_engine/examples/{swe_agent,retool,mle_dojo}`), so the trainer and
  evaluator see every token natively. The model is always served in-process
  inside the run's own container.
- **Installed-agent lane** — an external scaffold (OpenHands / Terminus-2) is
  installed black-box inside the task container. Any benchmark in Harbor task
  format runs through the shared infra in `rl_engine/examples/shared/`.

```text
installed lane: source benchmark -> Harbor-format task dirs
  -> tasks.jsonl + tests/ bundle -> rollout scheduler
  -> installed scaffold inside Rock sandbox
  -> tests/test.sh verifier reward -> eval JSONL or trainer update

in-house lane:  dataset JSONL -> rollout scheduler -> in-house agent loop
  (tool calls executed in Rock sandboxes) -> verifier reward
  -> eval JSONL or trainer update
```

Routing matrix — entry point and model serving per cell:

| Lane × mode | Entry point | Model serving | Section |
| --- | --- | --- | --- |
| Installed-agent eval | `shared/docker_run_eval.sh` | **external SGLang server** (`serve_sglang.sh`, `.venv-sglang`) | 4 |
| Installed-agent train | `shared/docker_run_train.sh` | in-process, inside the slime-rl image | 5 |
| In-house train | `scripts/train/swe/` recipes; ReTool `scripts/rl.sh` | in-process | 6.1, 6.2 |
| In-house eval | per-example `scripts/eval.sh` (SWE agent via `docker_launch.sh`) | in-process | 6.3 |

Important directories:

| Path | Purpose |
| --- | --- |
| `rl_engine/trainer/` | Training engine and algorithm recipes. |
| `rl_engine/rollout/` | Scheduler, evaluator routing, installed-agent runtime. |
| `rl_engine/examples/shared/` | Shared launchers: data prep, image build, eval, train, SGLang serve. |
| `rl_engine/examples/<bench>/` | Installed-lane benchmark integrations: config, evaluator profile, README. |
| `rl_engine/examples/{swe_agent,retool,mle_dojo}/` | In-house agent loops: agents, tools, sandbox clients, standalone eval scripts. |
| `rl_engine/examples/adapters/` | Local adapters from raw benchmark data to Harbor-format task dirs. |
| `env_engine/` | Rock sandbox service and CLI. |
| `scripts/` | SWE-agent end-to-end recipe scripts. |

Environment standard: two small pinned host uv envs (Rock, SGLang serving);
train and eval drivers run inside the slime-rl docker image. No conda envs,
no host RL venv.

- Rock env: `env_engine/.venv`, created with `uv sync --all-extras`, used by
  `rock admin start`. Host-only by necessity (spawns sibling containers).
- SGLang serving env: `.venv-sglang` at the repo root, pinned
  `sglang[all]==0.5.10rc0`, used by `serve_sglang.sh` for installed-agent
  eval only (Section 2.5).
- Train/eval env: the `lichangh20/slime-rl:stable` image, entered via
  `shared/docker_run_train.sh` / `shared/docker_run_eval.sh`.
- Repo env (`.venv` from `scripts/setup_env.sh`): optional, for host-side
  development and CPU tests only — not needed to train or eval.

Serving asymmetry (structural, not an accident): exactly one cell of the
matrix uses the external SGLang server — **installed-agent eval**. There
only token generation leaves the image, so any model a new-enough external
SGLang can serve (e.g. Qwen3.5) is evaluable long before it is trainable.
Every other cell — installed-agent train, in-house train, in-house eval —
is **self-contained**: it serves SGLang in-process on its own GPUs inside
the slime-rl image (training also syncs weights in-process), so the model
must be supported by the image's matched slime/SGLang/Megatron stack
(Qwen3 family; the image predates Qwen3.5, so Qwen3.5 training is
unsupported). The external server and `.venv-sglang` belong to
installed-agent eval alone; nothing else touches them.

## 1. Task Router

| If asked to... | Read/run |
| --- | --- |
| Set up a new machine | Section 2, then Section 3 for the needed data/model/task assets. |
| Evaluate a benchmark with an installed scaffold (OpenHands / Terminus-2) | Sections 2, 3.1, 3.2, and 4. Then the benchmark README under `rl_engine/examples/<bench>/README.md`. |
| Train with the shared installed-agent scaffold (SFT / GRPO) | Sections 2, 3.2, 3.3, and 5. Detail: headers of `rl_engine/examples/shared/train.sh` and `docker_run_train.sh`. |
| Run in-house SWE training recipes: GRPO, iter-SFT, OPD, DAgger, AggreVaTe | Sections 2.1, 2.3, 3.4, and 6.1. For mixed-loss DAgger/AggreVaTe, read `scripts/train/swe/QUICKSTART_OPD.md`. |
| Train the in-house ReTool math agent (GRPO) | Section 6.2, then `rl_engine/examples/retool/README.md`. |
| Evaluate a checkpoint with the in-house SWE agent (no training) | Sections 2.1, 2.3, 3.4, and 6.3. |
| Run MLE-Dojo or ReTool AIME eval (in-house) | Section 6.3, then `rl_engine/examples/{mle_dojo,retool}/README.md`. |
| Integrate a new benchmark | Section 7. Detail: `rl_engine/examples/adapters/SKILL.md`, then `rl_engine/examples/SKILL.md`. |
| Debug a failed run | Section 8 first, then the relevant launcher header and output JSONL/logs. |

Dependency note: every lane requires Rock admin, Docker with the slime-rl
image, and staged models/data. The installed-agent lane additionally needs
per-task Docker images and the site profile (Section 2.4), and
installed-agent *eval* needs the external SGLang server (Section 4.1). The
in-house lane needs none of those three: it serves the model in-process,
pulls per-instance sandbox images on demand, and does not read the site
profile.

## 2. One-Time Machine Setup

### 2.1 System Preconditions

Required for real eval/train:

- Linux host.
- Docker with permission for the current user. GPU workloads need NVIDIA
  container runtime.
- `uv`.
- CUDA GPUs for model serving/training.
- Enough disk for models, task Docker images, logs, and checkpoints.
- Network access for first-time dependency/model/task downloads.
- The repo cloned on local/scratch disk, not an NFS-mounted home. Containers
  run as root and write results and the HF cache under `external/`; NFS
  exports squash root to `nobody`, so those writes fail with
  `PermissionError`. The docker wrappers probe this and fail fast with
  instructions.

Recommended credentials:

```bash
docker login            # avoids Docker Hub pull limits during task-image builds
export WANDB_KEY=...    # required by SWE-agent recipe scripts; optional for eval/smokes
```

HF auth is optional — all referenced models and datasets are public (one
exception: the ReTool example expects a user-supplied checkpoint — Section
6.2). For HF downloads use `uvx --from 'huggingface_hub[cli]' hf ...` (no
host env needed; `huggingface-cli` is deprecated upstream).

Verify:

```bash
docker version                  # daemon reachable and you have permission
docker info | grep -i nvidia    # NVIDIA runtime registered (GPU containers work)
docker info | grep Username     # logged in
```

Hardware sizing guide:

| Workflow | Typical minimum |
| --- | --- |
| CPU-only import/dev tests | CPU, Docker optional. Run `scripts/setup_env.sh --cpu`. |
| Eval serving | Enough total GPU memory across the serving GPUs for model weights + KV cache at your context length; pick GPU count/TP accordingly. Reference: ~2 GB/B params for BF16 weights (~half for FP8), plus KV-cache headroom that grows with context and concurrency — e.g. a 30B-class FP8 model is ~35 GB of weights, so 1x80GB works at short context, 2x80GB is comfortable at 262k. Plus CPU/memory for sandboxes. |
| KernelBench eval | Serving GPUs plus sandbox GPUs in the cluster layout. |
| Shared installed-agent train, 4B default | 4 GPUs for the training job. |
| SWE-agent OPD/DAgger/AggreVaTe recipe scripts | 8 GPUs: teacher on 0-3, student on 4-7 by default. |
| Checkpoints | Plan tens of GB per checkpoint; Qwen3-4B with optimizer state is about 50 GB per saved step. |

### 2.2 Repo Environment (optional — dev only)

Train and eval do not need this env (they run inside the slime-rl docker
image; see the Section 0 environment standard). Install it only for host-side development,
unit tests, or CPU import checks:

```bash
bash scripts/setup_env.sh          # use --cpu for CPU-only dev setup
source activate_stacx.sh
```

Verify:

```bash
python -c "from rl_engine.trainer import create_trainer; print('rl_engine OK')"
python -c "import ray, torch; print('ray', ray.__version__, 'torch', torch.__version__)"
```

Details: `scripts/setup_env.sh` and `rl_engine/README.md`.

### 2.3 Rock Sandbox Environment

Start Rock in a separate terminal or tmux pane:

```bash
cd env_engine
uv venv --python 3.11 --python-preference only-managed
uv sync --all-extras
source .venv/bin/activate
rock admin start
```

Verify from another terminal:

```bash
curl http://127.0.0.1:8080/
```

Expected: HTTP 200 or a small Rock/admin response.

Optional sandbox smoke, still from the Rock env:

```bash
python - <<'PY'
import asyncio
from rock.actions import CreateBashSessionRequest
from rock.sdk.sandbox.client import Sandbox
from rock.sdk.sandbox.config import SandboxConfig

async def main():
    sb = Sandbox(SandboxConfig(image="python:3.11", memory="2g", cpus=1.0))
    await sb.start()
    await sb.create_session(CreateBashSessionRequest(session="bash-1"))
    r = await sb.arun(cmd="python --version", session="bash-1")
    print(r.output.strip())
    await sb.stop()

asyncio.run(main())
PY
```

Details: `env_engine/README_ROCK.md`.

### 2.4 Site Profile

The shared installed-agent eval/train scripts (Sections 4–5) read a site
profile from `rl_engine/examples/shared/profiles/`; the in-house lane
(Section 6) does not use it. The profile has two files:

- `<name>.env`: scratch root, SGLang IP, serving GPU ids, TP size.
- `cluster_layout_<name>.json`: sandbox resource budget and host IP.

Standard: the `default` profile holds the real values for this server.
Scripts fall back to `PROFILE=default` when `PROFILE` is unset, so runs on
this server never pass `PROFILE`. Only additional servers get a named pair
(`<name>.env` + `cluster_layout_<name>.json`) selected with `PROFILE=<name>`.

Setup is three steps, all edits to the default pair in place.

Step 1 — set the server IP in `cluster_layout_default.json`:

```bash
PROFILE_DIR=rl_engine/examples/shared/profiles
IP=$(hostname -I | awk '{print $1}')
sed -i "s/REPLACE_WITH_SERVER_IP/${IP}/" \
  "${PROFILE_DIR}/cluster_layout_default.json"
```

The key must be the LAN IP reachable from Docker containers, not
`127.0.0.1`. Skip if already set for this server.

Step 2 — set the sandbox budget in `cluster_layout_default.json`:

- `gpus`: sandbox GPU ids. Exclude the SGLang serving GPUs
  (`SGLANG_SERVE_GPUS` from step 3) and, on shared hosts, GPUs other
  users occupy (check `nvidia-smi`).
- `cpu` and `memory_gb`: scheduler budgets, not measured host capacity.
  Stay below `nproc` / MemTotal to leave headroom.

Step 3 — set the serving values in `default.env`:

- `SCRATCH_BASE`: fallback output root, used only when running `eval.sh`
  directly on the host without `EVAL_DETAILS_PATH` (output then lands in
  `${SCRATCH_BASE}/${USER}/<bench>/`). The major path — eval via
  `docker_run_eval.sh` (Section 4) — writes to
  `external/results/eval/...` instead and never reads this. Pick a large
  disk if you use the direct path.
- `PROFILE_SGLANG_IP`: host-reachable SGLang IP (auto-detects the first
  `hostname -I` entry; override if that is the wrong interface).
- `PROFILE_NUM_GPUS`: SGLang tensor parallel size — must match how the
  model is served. Pick it so total serving-GPU memory covers weights +
  KV cache (sizing reference: Section 2.1).
- `SGLANG_SERVE_GPUS`: GPU ids the model is served on (count =
  `PROFILE_NUM_GPUS`); keep them out of step 2's `gpus`.

Value flow at run time — the profile is the single source of truth:

- `PROFILE_NUM_GPUS` -> server TP (Section 4.1 / `serve_sglang.sh`) AND
  eval `--rollout-num-gpus-per-engine` (`eval.sh`). Both sides read the
  same value, so they match by construction.
- `SGLANG_SERVE_GPUS` -> the server process's `CUDA_VISIBLE_DEVICES`,
  i.e. which physical GPUs the model occupies.
- Layout `gpus`/`cpu`/`memory_gb` -> the sandbox scheduler budget.

Never hardcode GPU ids or TP in run commands; override per run only when
intentional (rules: end of Section 4).

Verify:

```bash
! grep -q REPLACE_WITH_SERVER_IP \
  rl_engine/examples/shared/profiles/cluster_layout_default.json
python -m json.tool \
  rl_engine/examples/shared/profiles/cluster_layout_default.json
```

To add another server, copy the pair to `<name>.env` +
`cluster_layout_<name>.json`, edit for that server, and pass
`PROFILE=<name>` on every command there. Named pairs are local files, not
meant to be committed.

Details: `rl_engine/examples/shared/profiles/README.md`.

### 2.5 SGLang Serving Environment (installed-agent eval only)

Installed-agent eval connects to an external SGLang server (Section 4.1).
In-house eval and all training serve the model in-process and never use
this env — skip this section when only running the in-house lane. The
server runs from a dedicated host uv env, pinned to the SGLang version
verified for Qwen3.5 serving. Create it once, at the repo root:

```bash
uv venv .venv-sglang --python 3.11
source .venv-sglang/bin/activate
uv pip install --prerelease=allow "sglang[all]==0.5.10rc0"
```

`--prerelease=allow` is required: this SGLang release pins a `flash-attn-4`
beta. Keep the exact version pin — the slime-rl image's SGLang predates
Qwen3.5, and unpinned installs are not reproducible. Do not install SGLang
into the rock env or a conda env.

Verify:

```bash
.venv-sglang/bin/python -c "import sglang; print(sglang.__version__)"
```

Expected: `0.5.10rc0`.

## 3. Stage Models, Data, and Task Trees

Model & HF cache standard — who downloads what, where:

- **Eval weights**: the host-side SGLang server (Section 4.1) downloads them
  as your user into your normal HF cache (`HF_HOME`, else
  `~/.cache/huggingface`). Nothing to configure. If your home dir is small or
  quota'd, `export HF_HOME=/scratch/$USER/.cache/huggingface` once before
  serving.
- **Eval container**: needs only `HF_CHECKPOINT`'s tokenizer/config (~MBs).
  Downloaded automatically on the first run into the repo-local cache
  `external/hf_cache` (mounted at `/root/.cache/huggingface`; override with
  `HOST_HF_CACHE`) and reused afterwards. Model weights never enter the eval
  container.
- **Train checkpoints**: always local dirs — `REF_LOAD` is a Megatron
  torch_dist directory with no hub form, so hub ids are not a supported
  `HF_CHECKPOINT` form for training. Stage host-side as your user:
  `uvx --from 'huggingface_hub[cli]' hf download <repo> --local-dir external/models/<name>`,
  then `HF_CHECKPOINT=/root/models/<name>` (per-recipe commands: Section 3.4).
- **Harbor task trees**: repo-local `external/harbor-datasets/<name>`
  (Section 3.1). Host-side only — never mounted into containers.

### 3.1 Harbor Task Trees

Standard: raw Harbor task trees stage in the repo-local
`external/harbor-datasets/<name>` — gitignored, host-side only, and on
local scratch by construction (the repo itself must live there). Every
bench uses the same location, so the download and
`prepare_jsonl.py --tasks-dir` commands chain verbatim.

Benchmarks in the public registry download with `download_tasks.py` (no
Harbor install required). List available datasets:

```bash
python rl_engine/examples/shared/download_tasks.py --list
```

Worked example — SWE-bench Verified, Django-100 subset:

```bash
python rl_engine/examples/shared/download_tasks.py swebench-verified@1.0 \
  -o external/harbor-datasets/swebench-verified_1.0 \
  --task-names-file rl_engine/examples/swebench/config/django100_task_names.yaml
```

The other benches follow the same pattern; each
`rl_engine/examples/<bench>/README.md` (Section 3.2 table) starts with its
exact download command. Two benches are not in the public registry and
generate their trees locally instead: `kernel_bench` (adapter,
`rl_engine/examples/adapters/kernelbench/`) and `skyrl` (HF dataset +
adapter, Section 3.3) — their READMEs cover it.

Task-names files (`--task-names-file` above, `--task-names-yaml` in
`prepare_jsonl.py`) share one format: a YAML `task_names:` block at any
indentation (a flat subset file like the one above, or a full Harbor run
config), or plain one-name-per-line text. The spec is
`download_tasks.read_task_names`, the single parser both scripts use; zero
parsed names is an error.

Verify a downloaded tree:

```bash
find external/harbor-datasets/swebench-verified_1.0 \
  -name instruction.md | wc -l
find external/harbor-datasets/swebench-verified_1.0 \
  -path '*/tests/test.sh' | wc -l
```

Expected for the Django-100 command: both counts are about 100.

### 3.2 Convert Task Trees and Build Images

Every eval/train task tree must become a repo-local `tasks.jsonl` plus sibling
`tests/` bundle, then Docker images must be built.

Example for SWE-bench Django-100:

```bash
python rl_engine/examples/shared/prepare_jsonl.py \
  --tasks-dir external/harbor-datasets/swebench-verified_1.0 \
  --output rl_engine/examples/swebench/config/tasks_django100.jsonl \
  --registry swebench \
  --task-names-yaml rl_engine/examples/swebench/config/django100_task_names.yaml

bash rl_engine/examples/shared/build_images.sh \
  rl_engine/examples/swebench/config/tasks_django100.jsonl
```

Verify:

```bash
wc -l rl_engine/examples/swebench/config/tasks_django100.jsonl
find rl_engine/examples/swebench/config/tests -path '*/test.sh' | wc -l
python - <<'PY'
import json
p = "rl_engine/examples/swebench/config/tasks_django100.jsonl"
row = json.loads(open(p).readline())
print(row["metadata"]["docker_image"])
print(row["metadata"]["tests_subdir"])
PY
```

Expected: `wc -l` is 100 for Django-100; `tests_subdir` is present. During
image build, expect `OK <task_id>` or `SKIP <task_id>`.

This section is the flow once, end to end, for one bench. Do not repeat it
per bench: each example README is the self-contained step-by-step
(download → prepare → build → run) for its benchmark:

| `BENCH` | Detail doc |
| --- | --- |
| `swebench` | `rl_engine/examples/swebench/README.md` |
| `terminal_bench` | `rl_engine/examples/terminal_bench/README.md` |
| `algotune` | `rl_engine/examples/algotune/README.md` |
| `kernel_bench` | `rl_engine/examples/kernel_bench/README.md` |
| `seta` | `rl_engine/examples/seta/README.md` |
| `skyrl` | `rl_engine/examples/skyrl/README.md` |

### 3.3 SkyRL Task Tree for Current Installed-Agent Training

`BENCH=skyrl` is the only train-enabled bench in `shared/train.sh` today.
Generate its task tree with the local adapter, then prepare JSONL and images.

```bash
uvx --from 'huggingface_hub[cli]' hf download lichangh20/stacx-skyrl-swe-train-293 \
  --repo-type dataset --local-dir external/data/swe

python rl_engine/examples/adapters/skyrl_293/run_adapter.py \
  --output-dir external/harbor-datasets/skyrl_293 \
  --data-path external/data/swe/skyrl_293.jsonl

python rl_engine/examples/shared/prepare_jsonl.py \
  --tasks-dir external/harbor-datasets/skyrl_293 \
  --output rl_engine/examples/skyrl/config/tasks.jsonl \
  --registry skyrl

bash rl_engine/examples/shared/build_images.sh \
  rl_engine/examples/skyrl/config/tasks.jsonl
```

Task-set standard (applies to every case, not just overfit): each task set is
defined by a **committed task-names YAML** (shared format, same as the Section
3.1/3.2 files) — the source of truth — while the JSONL and `tests/` bundle it
produces are **generated per host and gitignored**. The full set is the
no-filter case above (`tasks.jsonl`, all discovered tasks); every other case
is the same `prepare_jsonl.py` command plus `--task-names-yaml`. Rollout/eval
dataset YAMLs reference a generated JSONL by `path:` and set sampling policy
only — they do not define task membership.

Example — the overfit set used by the Section 5 smokes
(`config/overfit_task_names.yaml` -> `tasks_overfit.jsonl`):

```bash
python rl_engine/examples/shared/prepare_jsonl.py \
  --tasks-dir external/harbor-datasets/skyrl_293 \
  --output rl_engine/examples/skyrl/config/tasks_overfit.jsonl \
  --registry skyrl \
  --task-names-yaml rl_engine/examples/skyrl/config/overfit_task_names.yaml
```

To change an existing case (e.g. grow overfit to 3 tasks): edit its task-names
file and re-run the command — no yaml or code changes. To add a new case
(e.g. a 10-task debug set): commit `config/<case>_task_names.yaml`, generate
`tasks_<case>.jsonl` with the same command, and point the case's rollout/eval
yaml `path:` at it. The `tests/` bundle is shared (keyed by task id next to
the JSONLs) and per-task images are already covered by the full-set
`build_images.sh` above.

Verify:

```bash
wc -l rl_engine/examples/skyrl/config/tasks.jsonl
test -d rl_engine/examples/skyrl/config/tests
test -f rl_engine/examples/skyrl/config/tasks_overfit.jsonl
```

Expected: about 293 tasks, and one line per name in `overfit_task_names.yaml`
in the overfit JSONL (`0 skipped` on the prepare run).

### 3.4 Models and Data for SWE-Agent Recipe Scripts

The shared installed-agent training wrapper and SWE-agent recipe scripts expect
model directories mounted inside containers under `/root/models`, and data under
`/root/data`.

For the current `shared/docker_run_train.sh` defaults, stage the 4B instruct
checkpoint and Megatron dist checkpoint, then pass `HOST_MODELS`:

```bash
mkdir -p external/models external/data external/results external/ckpts external/logs

uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-4b-instruct-2507 \
  --local-dir external/models/qwen3-4b-instruct-2507
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-4b-instruct-2507-distcp \
  --local-dir external/models/qwen3-4b-instruct-2507_torch_dist
```

For the SWE-agent recipe scripts, also stage the SFT-iter0 student — the
default starting checkpoint of all five recipes (GRPO, iter-SFT, OPD, DAgger,
AggreVaTe), so cross-algorithm comparisons share one starting point — plus
the teacher and SWE data:

```bash
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-4b-instruct-sft-swegym-iter0 \
  --local-dir external/models/qwen3-4b-instruct-sft-iter0
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-4b-instruct-sft-swegym-iter0-distcp \
  --local-dir external/models/qwen3-4b-instruct-sft-iter0_torch_dist

uvx --from 'huggingface_hub[cli]' hf download Qwen/Qwen3-Coder-30B-A3B-Instruct \
  --local-dir external/models/qwen3-coder-30b-a3b-instruct

uvx --from 'huggingface_hub[cli]' hf download lichangh20/stacx-swe-online-dagger-data \
  --repo-type dataset --local-dir external/data/swe
```

For the 8B recipe variants (`*_8b.sh`), stage the 8B student pair the same
way (the HF repo names carry `-swegym-`; the local dir names match the
scripts' `BASE_HF` default `qwen3-8b-rope5m-64k-sft-iter0`):

```bash
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-8b-rope5m-64k-sft-swegym-iter0 \
  --local-dir external/models/qwen3-8b-rope5m-64k-sft-iter0
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-8b-rope5m-64k-sft-swegym-iter0-distcp \
  --local-dir external/models/qwen3-8b-rope5m-64k-sft-iter0_torch_dist
```

Recipe-specific optional assets:

```bash
# Iter-SFT only
uvx --from 'huggingface_hub[cli]' hf download lichangh20/stacx-iter-sft-corpus-qwen30bcoder \
  --repo-type dataset --local-dir external/data/iter_sft_corpus

# GRPO recipe script only
uvx --from 'huggingface_hub[cli]' hf download lichangh20/stacx-skyrl-swe-train-293 \
  --repo-type dataset --local-dir external/data/swe

# Optional post-hoc KL/JSD analysis
uvx --from 'huggingface_hub[cli]' hf download lichangh20/qwen3-coder-30b-swegym-train-eval-100-kl-cache \
  --repo-type dataset \
  --local-dir external/kl_cache/qwen30bcoder_swegym_train_eval_diverse
```

Verify common recipe-script files:

```bash
test -f external/data/swe/swe_gym_val_100.jsonl
test -f external/data/swe/swe_bench_django.jsonl
test -d external/models/qwen3-4b-instruct-sft-iter0
test -d external/models/qwen3-coder-30b-a3b-instruct
```

`scripts/prepare_models.sh` is useful for local dev/tests, but it is not a
complete substitute for the recipe-specific model names above.

## 4. Evaluate a Benchmark (Installed-Agent Lane)

This is the canonical shared eval path for installed scaffolds. It connects
to an external SGLang server. It does not launch one for you. (In-house
agent evals — SWE agent, MLE-Dojo, ReTool — are Section 6.3 instead.)

Preconditions:

- Section 2 complete.
- Rock admin is running and `curl http://127.0.0.1:8080/` works.
- The site profile exists and has no placeholder IP.
- The benchmark task JSONL and sibling `tests/` bundle exist.
- Task Docker images have been built.
- SGLang server is running and reachable from sandbox containers.

### 4.1 Start SGLang

Eval connects to an external OpenAI-compatible SGLang endpoint at
`SGLANG_ROUTER_IP:SGLANG_ROUTER_PORT`. The serving env is separate from the
train/eval image on purpose: newer models (e.g. Qwen3.5) need SGLang >=
v0.5.10rc0, which the slime-rl image predates. Serve from the pinned
`.venv-sglang` env (Section 2.5) via `serve_sglang.sh` — never from the
slime-rl image or ad-hoc conda envs (rationale: Section 0 environment
standard). Training does not use this server: it serves SGLang in-process
inside the image (Section 0 asymmetry note).

Standard: the profile is the single source of truth for serving GPUs and
TP (`SGLANG_SERVE_GPUS`, `PROFILE_NUM_GPUS` — Section 2.4). Never hardcode
GPU ids or TP in a serve command; `serve_sglang.sh` reads them from the
profile so the server and eval agree by construction. Run in its own
terminal, from the repo root:

```bash
source .venv-sglang/bin/activate
bash rl_engine/examples/shared/serve_sglang.sh
```

Defaults: model `Qwen/Qwen3.5-35B-A3B-FP8` (eval's `HF_CHECKPOINT`
default), port 9001 (also the eval default; the canonical eval command
still sets `SGLANG_ROUTER_PORT` explicitly so the server/eval pairing
stays visible). Wait for "The server is fired up and ready to roll!". Per-run
overrides go through the script's env knobs, e.g.
`MODEL=... PORT=9002 SGLANG_SERVE_GPUS=2,3 SGLANG_TP_SIZE=2` (full list in
the `serve_sglang.sh` header).

Verify from another terminal:

```bash
curl http://$(hostname -I | awk '{print $1}'):9001/v1/models
```

Expected: JSON model list and HTTP 200.

### 4.2 Run SWE-bench Django-100 Eval

The eval driver runs inside the slime-rl image via the thin wrapper
`shared/docker_run_eval.sh` (same pattern as `docker_run_train.sh`; eval
logic stays in `eval.sh`, but always launch through the wrapper). Canonical
form — name the bench, the scaffold, and the server port:

```bash
BENCH=swebench \
SCAFFOLD=openhands \
SGLANG_ROUTER_PORT=9001 \
  bash rl_engine/examples/shared/docker_run_eval.sh
```

Results default to `external/results/eval/<BENCH>_<SCAFFOLD>_<timestamp>.jsonl`;
the wrapper banner prints the exact path (`Output:`). Set `EVAL_DETAILS_PATH`
only to write somewhere else. (Direct host-side `eval.sh` runs — not the
normal path — fall back to `${SCRATCH_BASE}/${USER}/<bench>/` from the
profile instead.) Every other bench runs the same way — see
`rl_engine/examples/<bench>/README.md` for its prep commands and
bench-specific knobs.

The wrapper returns immediately — follow the run with the printed `tail -f`
command and verify after it completes.

Verify (on the wrapper-printed output path):

```bash
OUT=$(ls -t external/results/eval/swebench_openhands_*.jsonl | head -1)
wc -l "$OUT"
ls "${OUT%.jsonl}.summary.json"
python - "$OUT" <<'PY'
import json, sys
rows = [json.loads(l) for l in open(sys.argv[1]) if l.strip()]
print("rows", len(rows))
rewards = {r.get("reward") for r in rows}
print("reward values", sorted(v for v in rewards if v is not None)[:5],
      "(+ None)" if None in rewards else "")
PY
```

Expected: JSONL rows equal the eval dataset size; `.summary.json` exists. The
trainer log also prints `eval/<dataset>/accuracy`.

Run outputs — everything an eval persists sits next to the results JSONL:

- `<name>.jsonl` — per-task result rows; `<name>.summary.json` — aggregate.
- `container_logs/<name>/<task_id>/agent/` — per-task sandbox logs,
  bind-mounted as `/logs` into each task container while it runs:
  `openhands.txt` (scaffold CLI stdout), `openhands.trajectory.json`,
  `completions/*.json` (raw LLM request/response bodies). The JSONL filename
  stem namespaces the tree, so reruns never clobber earlier logs;
  `CONTAINER_LOG_DIR` overrides the location. This mount exists only when an
  eval-details path is set (always true via the wrapper); train-rollout task
  containers get no host log mount. Train-time evals do get it: their
  eval-details path keeps a `{rollout_id}` placeholder (one launch, many
  evals), so logs land per iteration at
  `train_runs/<run>/eval_details/container_logs/<rollout label>/<task_id>/`
  (e.g. `train_1`, resolved from `STACX_ROLLOUT_ID` at container start).
- `EVAL_DETAILS_PATH` defaults to the timestamped
  `external/results/eval/<bench>_<scaffold>_<ts>.jsonl` above — the normal
  case; leave it unset. Override only when the run must write elsewhere
  (the filename stem also renames the `container_logs/` tree).
- Both Docker wrappers run detached: they print the banner and the follow-up
  handles, then return immediately. The driver stream goes to a log that
  survives the `--rm` container — eval: `<EVAL_DETAILS_PATH stem>.log` next
  to the results JSONL; train: `HOST_RESULTS/train/<container name>.log`.
  `tail -f` it to follow the run (the wrapper prints the exact command).
  Success/failure is read from the log tail (train also: checkpoints under
  `SAVE_PATH`; eval also: `.summary.json`), not the wrapper's exit code.

Path rule for overrides. Passthrough vars naming a file (`EVAL_CONFIG`,
`CLUSTER_LAYOUT` — same for the train wrapper's equivalents) are read
*inside* the container, where the repo is mounted at `/root/stacx_eval` and
the driver runs from that directory. Pass them as repo-relative paths
(valid on host and container alike) or as `/root/stacx_eval/...`; a
host-absolute path like `$PWD/...` or `/scratch/...` does not exist in the
container and fails with `FileNotFoundError`. The same applies to paths
*inside* a dataset yaml (`path:` entries) — keep those repo-relative too.
Two exceptions: paths under `HOST_RESULTS` (default `external/results`) are
same-path mounted, so host paths work there — which is why the
`EVAL_DETAILS_PATH` default is fine; and the `HOST_*` knobs themselves
(`HOST_MODELS`, `HOST_RESULTS`, `HOST_HF_CACHE`) are wrapper-only and always
take host paths.

Common eval knobs:

| Env var | Meaning |
| --- | --- |
| `BENCH` | Example dir under `rl_engine/examples/`. Required. |
| `SCAFFOLD` | `openhands` or `terminus2`. Default `openhands`. |
| `PROFILE` | Site profile. Default `default` (this server); set only for another server's named profile. |
| `EVAL_CONFIG` | Dataset YAML. Defaults to the benchmark's `config/eval_datasets.yaml`. Repo-relative path (see the path rule above). |
| `EVALUATOR_CONFIG_PATH` | Builder import path. Defaults to `build_evaluators_${SCAFFOLD}`. |
| `HF_CHECKPOINT` | Model name for the rollout process. Keep it the hub id the server serves — it doubles as `AGENT_MODEL`, the name scaffolds send to the router. Only its tokenizer/config load in-container (cached in `external/hf_cache`). |
| `HOST_HF_CACHE` | Host dir for the container HF cache. Default `external/hf_cache`; must be root-writable (Section 2.1). |
| `SGLANG_ROUTER_IP` / `SGLANG_ROUTER_PORT` | Model server address. Defaults: profile IP, port 9001. |
| `NUM_GPUS` | Rollout GPUs per engine; must equal the server's TP. Defaults to `PROFILE_NUM_GPUS`. |
| `CONCURRENCY` | Max in-flight samples for the evaluator. Defaults to the benchmark profile's `concurrency` (its `evaluator_config.py`). On GPU benches, keep `CONCURRENCY x gpus_per_sample <=` the layout `gpus` count (see the GPU-bench invariant below). |
| `EVAL_DETAILS_PATH` | Per-sample JSONL output path. Default `external/results/eval/<bench>_<scaffold>_<ts>.jsonl`; set only to redirect output. Its filename stem also names the `container_logs/` tree (see Run outputs above). |
| `APPEND_ONLY` | `auto`, `on`, or `off`. Token capture requires append-only. |

Override rules. Every profile value is only a default — explicit env wins.
The default run overrides nothing; when you do override (e.g. a second
server + eval pair on the same host), keep these invariants or hit the
Section 8 mismatch failures:

- Server TP == eval `NUM_GPUS` per endpoint. Only the id count in
  `SGLANG_SERVE_GPUS` vs TP is enforced at launch (`serve_sglang.sh`);
  the TP-vs-eval match is not enforced anywhere — check it yourself.
- Serving GPU ids stay out of the cluster layout `gpus` — but this only
  matters when the bench schedules sandbox GPUs (e.g. KernelBench);
  non-GPU benches ignore the list. If a GPU bench needs a temporary
  different layout, pass `CLUSTER_LAYOUT=<path to a json>` per run
  instead of editing the profile.
- GPU benches take sandbox GPUs from the layout `gpus` list. A bench is
  a GPU bench when its `evaluator_config.py` sets `gpus_per_sample > 0`
  (today only KernelBench: 1 GPU per task, `concurrency=3`). The
  evaluator's declared demand is `CONCURRENCY x gpus_per_sample`, and
  the scheduler asserts it fits the layout at startup — exceeding the
  `gpus` count aborts immediately with a `Deadlock: ... placement
  tokens` error rather than throttling. So whenever you shrink the GPU
  list (profile edit or per-run `CLUSTER_LAYOUT`), lower `CONCURRENCY`
  to match: `CONCURRENCY x gpus_per_sample <= len(gpus)`. The reverse
  is safe — fewer in-flight tasks than GPUs just leaves GPUs idle.
- Each concurrent server gets its own port; point its eval at it with
  `SGLANG_ROUTER_PORT`.
- Concurrent runs each debit their own copy of the layout `cpu`/`memory_gb`
  budget — nothing coordinates total load across simultaneous runs, so
  leave headroom when running several.

## 5. Train with the Shared Installed-Agent Scaffold

Use this path for shared installed-agent scaffold training: `BENCH x EVAL_BENCH x SCAFFOLD x ALGO`.

Preconditions:

- Sections 2 and 3.3 complete.
- `HOST_MODELS` points to a host dir containing the checkpoint names used by
  `HF_CHECKPOINT` and `REF_LOAD` — locally staged dirs per the Section 3
  model standard (the wrapper prints the staging command if the dir is
  missing).
- `HOST_DATA` points to staged training/eval JSONL data when needed.
- Rock admin is running.
- Docker can pull/run `lichangh20/slime-rl:stable` unless `IMAGE` overrides it.
- Site profile and cluster layout are valid.

The wrapper returns immediately (driver log:
`HOST_RESULTS/train/<container name>.log`) — for every recipe below, follow
the run with the printed `tail -f` command and verify after it completes.

Supported axes in `rl_engine/examples/shared/train.sh`:

| Axis | Current values |
| --- | --- |
| `BENCH` | `skyrl` only. |
| `EVAL_BENCH` | Any example with `config/eval_datasets.yaml` and scaffold builders. Default `swebench`. |
| `SCAFFOLD` | `openhands`, `terminus2`. |
| `ALGO` | `sft`, `grpo`. `opd` is reserved but not wired here. |

Routing invariant: dataset `task_type` must equal the evaluator
`BenchmarkProfile.evaluator_id`. Train rollouts use a capture evaluator;
eval uses a plain evaluator when `EVAL_BENCH != BENCH`.

### 5.1 SFT Smoke

Run a one-task smoke before full training:

```bash
HOST_MODELS=$PWD/external/models \
HOST_DATA=$PWD/external/data \
HOST_RESULTS=$PWD/external/results \
BENCH=skyrl \
EVAL_BENCH=skyrl \
ALGO=sft \
SCAFFOLD=openhands \
ROLLOUT_CONFIG=rl_engine/examples/skyrl/config/sft_rollout_overfit_smoke.yaml \
EVAL_CONFIG=rl_engine/examples/skyrl/config/rl_eval_overfit.yaml \
NUM_ROLLOUT=1 \
ROLLOUT_BATCH_SIZE=1 \
GLOBAL_BATCH_SIZE=1 \
SKYRL_MAX_TURNS=1 \
OPENHANDS_MAX_OUTPUT_TOKENS=1024 \
SKIP_EVAL_BEFORE_TRAIN=1 \
EVAL_INTERVAL=999999 \
USE_WANDB=0 \
  bash rl_engine/examples/shared/docker_run_train.sh
```

Verify:

```bash
find external/results -path '*checkpoints*' -o -path '*train_runs*' | head
find external/results -name 'resume_state.json' -print
```

Expected: the Ray job launches, one rollout/training pass completes, and a
`resume_state.json` appears under the run checkpoint directory.

### 5.2 GRPO Smoke

```bash
HOST_MODELS=$PWD/external/models \
HOST_DATA=$PWD/external/data \
HOST_RESULTS=$PWD/external/results \
BENCH=skyrl \
EVAL_BENCH=skyrl \
ALGO=grpo \
SCAFFOLD=openhands \
ROLLOUT_CONFIG=rl_engine/examples/skyrl/config/rl_rollout_overfit.yaml \
EVAL_CONFIG=rl_engine/examples/skyrl/config/rl_eval_overfit.yaml \
NUM_ROLLOUT=3 \
ROLLOUT_BATCH_SIZE=1 \
GLOBAL_BATCH_SIZE=8 \
SKIP_EVAL_BEFORE_TRAIN=1 \
EVAL_INTERVAL=999999 \
USE_WANDB=0 \
  bash rl_engine/examples/shared/docker_run_train.sh
```

### 5.3 Full Default Runs

SFT on SkyRL-293, eval on SWE-bench:

```bash
HOST_MODELS=$PWD/external/models \
HOST_DATA=$PWD/external/data \
HOST_RESULTS=$PWD/external/results \
BENCH=skyrl \
EVAL_BENCH=swebench \
SCAFFOLD=openhands \
ALGO=sft \
  bash rl_engine/examples/shared/docker_run_train.sh
```

GRPO on the same axes:

```bash
HOST_MODELS=$PWD/external/models \
HOST_DATA=$PWD/external/data \
HOST_RESULTS=$PWD/external/results \
BENCH=skyrl \
EVAL_BENCH=swebench \
SCAFFOLD=openhands \
ALGO=grpo \
  bash rl_engine/examples/shared/docker_run_train.sh
```

Progress checks:

```bash
find external/results -name resume_state.json -print
find external/results -path '*eval_details*' -type f | head
```

Detail: headers of `rl_engine/examples/shared/train.sh` and
`rl_engine/examples/shared/docker_run_train.sh`.

## 6. In-House Agent Lane: Train Recipes and Standalone Eval

The in-house lane's agent loops live in this repo
(`rl_engine/examples/{swe_agent,retool,mle_dojo}`). Every run here is
self-contained: the launcher starts a container that serves SGLang
in-process on its own GPUs — no external SGLang server, no `.venv-sglang`,
no site profile, no per-task image builds (per-instance sandbox images are
pulled from Docker Hub on demand, which is why `docker login` matters).

### 6.1 SWE Training Recipe Scripts

Use these scripts when asked for the SWE-agent recipe mode. This is a
supported training path alongside the shared installed-agent scaffold in Section 5.

For mixed-loss DAgger-OPD and AggreVaTe, use
`scripts/train/swe/QUICKSTART_OPD.md` as the detailed newcomer guide. It
covers the 4B/8B launch scripts, model/data staging, Rock, output roots,
knobs, result layout, gotchas, and resume semantics. For GRPO, iter-SFT, and
standalone OPD, read the selected script header before launching — the
headers document the algorithm, data prerequisites, GPU layout, and every
override knob.

Preconditions common to all:

- Repo env is available or the script's Docker image provides runtime deps.
- Rock admin is running on `127.0.0.1:${ROCK_PORT:-8080}`.
- `docker pull lichangh20/slime-rl:stable` succeeds, unless `SLIME_IMAGE`
  overrides it.
- `external/models/<BASE_HF>/` and `external/models/<BASE_HF>_torch_dist/`
  exist.
- `external/data/swe/` contains the recipe's JSONL files.
- `WANDB_KEY` is exported (this exact name — the scripts do not read
  `WANDB_API_KEY`).
- Pre-create `external/{ckpts,logs,results,data,models}` to avoid root-owned
  directories from Docker.

Recipe table — per-recipe footprint and the Section 3.4 assets it needs:

| Recipe | Script | GPUs | Teacher | Extra assets beyond the common 3.4 staging |
| --- | --- | --- | --- | --- |
| GRPO, reward-only on-policy RL | `scripts/train/swe/grpo/grpo_{4b,8b}.sh` | 8 by default (TP=8) | none — frozen ref for KL only | `external/data/swe/skyrl_293*.jsonl` (3.4 "GRPO recipe script only" block) |
| Iterative SFT on saved teacher trajectories | `scripts/train/swe/sft/sft_{4b,8b}.sh` | 4 (default 0-3) | none — replays saved corpus | `external/data/iter_sft_corpus/` (3.4 "Iter-SFT only" block) |
| OPD, reverse-KL vs teacher | `scripts/train/swe/opd/opd_{4b,8b}.sh` | 8: teacher 0-3, student 4-7 | 30B live | common 3.4 set |
| DAgger + OPD, beta-mixture rollin | `scripts/train/swe/online_dagger/dagger_{4b,8b}.sh` | 8: teacher 0-3, student 4-7 | 30B live | common 3.4 set; guide: `QUICKSTART_OPD.md` |
| DAgger + OPD, forced-expert tail/AggreVaTe | `scripts/train/swe/online_dagger/aggrevate_{4b,8b}.sh` | 8: teacher 0-3, student 4-7 | 30B live | common 3.4 set; guide: `QUICKSTART_OPD.md` |

Recipe-specific notes (verified against the script headers):

- **GRPO** starts from the same `…-sft-iter0` checkpoint as the other four
  recipes (fair-comparison default); its `--ref-load` KL anchor is that same
  frozen starting checkpoint. Set `BASE_HF=qwen3-4b-instruct-2507` (4B) /
  `BASE_HF=qwen3-8b-rope5m-64k` (8B) for the raw base instead (wider
  exploration, no SWE tool-call format training). It defaults to all 8
  GPUs with `TENSOR_PARALLEL=8`; to run on fewer GPUs set **both**
  `STUDENT_GPUS=<ids>` and `TENSOR_PARALLEL=<count>` — changing only one
  launches a broken TP/DP split. Example 4-GPU launch:
  `WANDB_KEY=<key> STUDENT_GPUS=0,1,2,3 TENSOR_PARALLEL=4 bash scripts/train/swe/grpo/grpo_4b.sh`.
- **Iter-SFT** bypasses live rollout: each iter loads
  `${CORPUS_DIR}/trajectory_sft_samples_*.pt` (default corpus:
  `external/data/iter_sft_corpus`). No teacher server; outputs land under
  `external/{ckpts,logs,results/eval}/baselines/iter_sft/`.
- **Cache location**: GRPO and iter-SFT default `CACHE_ROOT` to
  `~/.cache/stacx_{grpo,iter_sft}`. On hosts whose home is NFS (Section 2.1
  rule), set `CACHE_ROOT=$PWD/external/cache/<name>` — otherwise Triton hits
  `PermissionError` during CUDA-graph capture minutes in and SGLang dies
  (Section 8).

Launch examples:

```bash
WANDB_KEY=$WANDB_KEY bash scripts/train/swe/grpo/grpo_4b.sh
WANDB_KEY=$WANDB_KEY bash scripts/train/swe/sft/sft_4b.sh
WANDB_KEY=$WANDB_KEY bash scripts/train/swe/opd/opd_4b.sh
WANDB_KEY=$WANDB_KEY bash scripts/train/swe/online_dagger/dagger_4b.sh
WANDB_KEY=$WANDB_KEY bash scripts/train/swe/online_dagger/aggrevate_4b.sh
```

Progress checks:

```bash
find external/ckpts -name resume_state.json -print
find external/results -path '*eval_details*' -type f | head
find external/logs -type f | tail
```

These scripts auto-resume from `resume_state.json` when relaunched with the same
run tag/defaults. Do not change `RUN_TAG`, model path, or save path unless you
intend to start a separate run.

### 6.2 ReTool: Tool-Calling Math GRPO

A multi-turn agent solves DAPO-math problems by emitting Python tool calls
executed in a Rock sandbox, trained with GRPO. Predates the SWE lanes; kept
as a reference implementation.

The full walkthrough is `rl_engine/examples/retool/README.md`: stage the
data (`bash rl_engine/prompts/retool/load_dataset.sh`, mounted at
`/root/data/dapo-math-17k/`), start a slime container with the exact
`docker run` from that README — it carries the required
`--ulimit nofile=524288:524288` (Section 8) — then run
`scripts/rl.sh` inside via `docker exec` with `WANDB_API_KEY` set. Default
model pair is `/root/models/qwen3-4b-instruct-2507` plus
`/root/models/qwen3-4b-instruct-2507_torch_dist`; knobs (`HF_CHECKPOINT`,
`REF_LOAD`, `SAVE_DIR`, `MODEL_CONFIG`) are in the script header.

### 6.3 Standalone In-House Eval (no training)

Each in-house example ships an eval script driving the eval-only path of
`rl_engine.train` (`--num-rollout 0`) — the same agent loop the training
recipes use, so a trained checkpoint evaluates in exactly the loop it was
trained in. The container serves the model itself on the GPUs you give it;
do **not** start `serve_sglang.sh` for these.

**SWE agent** — evaluate a checkpoint on SWE-Gym / SWE-bench. Launch from
the host via the wrapper (it starts the container, which runs
`eval_multi_run.sh` -> `eval.sh` inside):

```bash
HOST_CACHE=$PWD/external/cache \
DATASET=django_100 GPUS=0,1,2,3 \
  bash rl_engine/examples/swe_agent/scripts/docker_launch.sh swe-agent
```

Preconditions and knobs:

- Rock admin running (Section 2.3); eval JSONLs staged under
  `external/data/swe/` (Section 3.4).
- The model needs **both** dirs under `external/models/`: the HF dir and
  `<name>_torch_dist`. Defaults (in `eval.sh`):
  `qwen3-coder-30b-a3b-instruct{,_torch_dist}` at TP=4.
- `DATASET` shorthands: `django_20` | `django_100` | `verified` (mapped to
  `/root/data/swe/*.jsonl`). `swe_bench_django_20.jsonl` is not in the HF
  dataset bundle — generate it once:
  `head -20 external/data/swe/swe_bench_django.jsonl > external/data/swe/swe_bench_django_20.jsonl`.
  `swe_bench_verified.jsonl` (the `verified` shorthand) is likewise
  generated, not downloaded — once, from the repo root:
  `bash rl_engine/prompts/swe/load_dataset.sh` (pulls the public
  princeton-nlp/SWE-bench_Verified parquet next to the script), then
  `uv run --no-project --with pandas --with pyarrow python rl_engine/prompts/swe/build_swebench.py`
  writes `external/data/swe/swe_bench_verified.jsonl` (500 tasks).
- `HOST_CACHE` defaults to `~/.cache/stacx_eval` — on NFS-home hosts set it
  repo-local as above (Section 2.1 rule; same Triton failure mode as
  Section 6.1's `CACHE_ROOT`).
- Container-side knobs (`HF_CHECKPOINT`, `REF_LOAD`, `MODEL_SCRIPT`,
  `NUM_GPUS` = TP) are read by `eval.sh` *inside* the container; the wrapper
  does not forward them — pass them through
  `EXTRA_DOCKER_ARGS='-e HF_CHECKPOINT=/root/models/<name> -e REF_LOAD=/root/models/<name>_torch_dist'`
  (container paths: `external/models` mounts at `/root/models`).
- Parallel runs: distinct `GPUS` plus distinct `RAY_*_PORT`s — the wrapper
  prints a worked example.

Verify: the wrapper prints the `tail -f` log command and a results check.
Results land under `external/results/eval_swe_agent/<run dir>/eval_details/`
(one JSONL per pass; count `reward > 0` rows for resolved). Aggregate with
`rl_engine/examples/swe_agent/analyze/analyze_eval.py`. Agent internals
(tools, sandbox client, grading router): `rl_engine/examples/swe_agent/README.md`.

**MLE-Dojo** — Kaggle-competition ML engineering eval in GPU sandboxes.
Needs an MLE-Dojo checkout + competition data and three host-path env vars
(`MLE_DATA_DIR_LOCAL`, `MLE_OUTPUT_DIR_LOCAL`, `MLE_ENTRYPOINT_LOCAL`);
`scripts/eval.sh` runs inside the slime container and fails fast without
them. Walkthrough: `rl_engine/examples/mle_dojo/README.md`.

**ReTool AIME** — same container pattern as Section 6.2 with
`scripts/eval.sh` instead of `rl.sh`. Walkthrough:
`rl_engine/examples/retool/README.md`.

## 7. Integrate a New Benchmark

Benchmark onboarding is two steps. STACX uses Harbor task format, but the Harbor
package/CLI is not required.

### 7.1 Step 1: Get a Harbor-Format Task Tree

First check whether the benchmark is already registered:

```bash
python rl_engine/examples/shared/download_tasks.py --list
```

If registered, download it with `download_tasks.py` and skip adapter writing.

If not registered, write a local adapter under
`rl_engine/examples/adapters/<adapter-id>/`. Use:

```text
rl_engine/examples/adapters/SKILL.md
```

Adapter output contract, one directory per task:

```text
instruction.md
task.toml
environment/Dockerfile
tests/test.sh
```

Verifier contract:

- `tests/test.sh` must always write one bare float reward to
  `/logs/verifier/reward.txt`, including error paths.
- Put test-only dependencies in `tests/test.sh`, not in the image, unless they
  are huge or required at build time.
- Keep `instruction.md`, Dockerfile `WORKDIR`, and verifier assumptions
  consistent.

Verify adapter output:

```bash
python rl_engine/examples/adapters/<adapter-id>/run_adapter.py \
  --output-dir external/harbor-datasets/<adapter-id> --limit 2

find external/harbor-datasets/<adapter-id> -name instruction.md | wc -l
find external/harbor-datasets/<adapter-id> -path '*/tests/test.sh' | wc -l
```

### 7.2 Step 2: Add the STACX Example

Use:

```text
rl_engine/examples/SKILL.md
```

Required example layout:

```text
rl_engine/examples/<bench>/
  evaluator_config.py
  README.md
  config/eval_datasets.yaml
  config/tasks.jsonl
  config/tests/<task_id>/...
```

Key invariant:

```text
BenchmarkProfile.evaluator_id == task_type in config/eval_datasets.yaml
```

Build and run:

```bash
python rl_engine/examples/shared/prepare_jsonl.py \
  --tasks-dir external/harbor-datasets/<adapter-id> \
  --output rl_engine/examples/<bench>/config/tasks.jsonl \
  --registry <evaluator_id>

bash rl_engine/examples/shared/build_images.sh \
  rl_engine/examples/<bench>/config/tasks.jsonl

BENCH=<bench> SCAFFOLD=openhands SGLANG_ROUTER_PORT=9001 \
  bash rl_engine/examples/shared/docker_run_eval.sh
```

Use the existing examples as references:

- No custom classes: `terminal_bench`, `swebench`.
- Artifact capture/custom agent: `algotune`, `kernel_bench`, `seta`.
- Train capture builders: `skyrl`, `seta`.

## 8. Failure Index

| Symptom | Likely cause | Check/fix |
| --- | --- | --- |
| `rock not healthy`, connection refused | Rock admin not running | Start Section 2.3, then `curl http://127.0.0.1:8080/`. |
| Eval says cluster layout missing | Default profile not set up, or `PROFILE` names a missing pair | Section 2.4; ensure `cluster_layout_<profile>.json` exists (default profile: `cluster_layout_default.json`). |
| Placeholder IP error | `cluster_layout_<profile>.json` still has `REPLACE_WITH_SERVER_IP` | Replace with LAN IP. Do not use `127.0.0.1`. |
| Containers cannot reach SGLang | `PROFILE_SGLANG_IP` or `SGLANG_ROUTER_IP` not container-reachable | Use host LAN IP; verify `curl http://IP:PORT/v1/models`. |
| SGLang external engine mismatch | Serving TP/GPU ids do not match eval/train args | Align `PROFILE_NUM_GPUS`, `SGLANG_SERVE_GPUS`, `NUM_GPUS`, and `ROLLOUT_GPUS_PER_ENGINE`. |
| `Deadlock: EvaluatorSpec ... needs N placement tokens but the pool can only provide M` | Eval memory capacity cannot support the requested concurrency (e.g. 64 Terminus2 samples × 8 GB = 512 GB; a 384 GB pool permits 48). | After confirming server headroom, increase the profile's `memory_gb`; otherwise lower `CONCURRENCY` to the pool capacity. |
| Train step `torch.OutOfMemoryError` in the log-prob/entropy loss (`_VocabParallelEntropy`), often after hours of healthy rollouts | One long-tail rollout sample. Loss memory is ~0.5–0.6 MB/token (fp32 vocab-shard logits, 4B TP=4), so a sample past ~64k total tokens OOMs an 80 GB GPU; `--max-tokens-per-gpu` packs micro-batches but cannot split a single sample, and extra data-parallel GPUs do not help. | Lower the dataset's `max_context_len` in the rollout YAML (forwarded to the scaffold's input cap; trained sample ≈ it + `max_response_len`, so e.g. 32768 keeps the worst case ≤ ~64k). Confirm the crash-adjacent `rollout/response_len/max` in the log exceeds what fit before. |
| `no evaluator for task_type` | `task_type` does not equal `BenchmarkProfile.evaluator_id` | Fix `config/eval_datasets.yaml` or `evaluator_config.py`. |
| Missing tests bundle | Ran with stale/moved JSONL or skipped `prepare_jsonl.py` | Ensure JSONL has `metadata.tests_subdir` and sibling `config/tests/`. |
| Build script cannot find Dockerfile | `metadata.task_dir` points to missing task tree | Re-run `prepare_jsonl.py` on this machine or keep the source task tree. |
| Docker Hub pull limit | Anonymous Docker pulls | Run `docker login`; pre-pull common images. |
| Root-owned `external/` dirs | Docker created missing bind-mount dirs as root | Pre-create dirs as your user. Fix an existing mess with `docker run --rm -v $(pwd)/external:/mnt alpine chmod -R 777 /mnt/ckpts /mnt/logs /mnt/results`. |
| Ray crashes with socket/path length | Long temp path/run tag | Use shorter `TMPDIR`, `RUN_NAME`, or script defaults that hash Ray tmp. |
| Port collision | Old Ray/SGLang process still running | Pick new ports or stop stale processes. Avoid broad `pkill` on shared hosts. |
| `ray job submit` driver dies ~30s in: "Failed to get the system config from raylet because it is dead ... timed out before receiving SETTINGS frame" | Hand-rolled `docker run` without ulimits: default `nofile=1024` starves the raylet of fds on many-core hosts (`Too many open files` in `raylet.err`); gRPC handshakes then hang. Looks like a network/IP problem but is not | Launch containers with `--ulimit nofile=524288:524288 --ulimit memlock=-1 --ulimit stack=67108864` (as `shared/docker_run_train.sh` does), or use the shared launchers. Confirm with `ulimit -n` inside the container. |
| `cannot set both Count and DeviceIDs on device request` | Multi-id GPU list (`GPUS=1,2,3,4` or similar) reached docker's CSV `--gpus` parser without literal quotes, so bare ids read as a Count | Docker needs `--gpus '"device=1,2,3,4"'` — the quotes must survive any `eval`/re-parse in the launcher (applies to `swe_agent/scripts/docker_launch.sh` and any script passing `--gpus device=` lists). |
| `ERROR: ... SWE_ENTRYPOINT_LOCAL is not set` | In-house SWE eval's `eval.sh`/`eval_multi_run.sh` run directly without the host path to `execute_sandbox.py` | Launch via `swe_agent/scripts/docker_launch.sh` (sets it), or export `SWE_ENTRYPOINT_LOCAL=<host path>/rl_engine/examples/swe_agent/environment/execute_sandbox.py`. |
| Triton `PermissionError` during CUDA-graph capture minutes into an in-house run; SGLang dies, 0 results | `HOST_CACHE`/`CACHE_ROOT` default under `~/.cache/` and home is NFS root-squashed (Section 2.1) | Point them at a repo-local dir, e.g. `HOST_CACHE=$PWD/external/cache` (eval) / `CACHE_ROOT=$PWD/external/cache/<name>` (recipes). |
| Started `serve_sglang.sh` for an in-house train/eval, run ignores it or ports collide | Wrong lane — only installed-agent eval uses the external server (Section 0 matrix) | Skip the server; in-house runs serve in-process on the GPUs passed to the launcher (Section 6). |
| `ALGO=opd` fails in `shared/train.sh` | OPD not wired for current scaffold train path | Use the Section 6.1 OPD recipe (`scripts/train/swe/QUICKSTART_OPD.md`). |
| Eval JSONL has many infra errors | Task images, tests, or sandbox resources are wrong | Inspect `container_logs/`, `.summary.json`, and task `reward.txt`/verifier output. |

## 9. Quick Reference Commands

Import check:

```bash
source activate_stacx.sh
python -c "from rl_engine.trainer import create_trainer; print('OK')"
```

Rock check:

```bash
curl http://127.0.0.1:8080/
```

Serve SGLang (own terminal; env from §2.5):

```bash
source .venv-sglang/bin/activate
bash rl_engine/examples/shared/serve_sglang.sh
```

SGLang check:

```bash
curl http://$(hostname -I | awk '{print $1}'):9001/v1/models
```

List public task datasets:

```bash
python rl_engine/examples/shared/download_tasks.py --list
```

Run installed-agent eval (driver in docker; needs the SGLang server above):

```bash
BENCH=swebench SCAFFOLD=openhands SGLANG_ROUTER_PORT=9001 \
  bash rl_engine/examples/shared/docker_run_eval.sh
```

Run installed-agent scaffold train:

```bash
HOST_MODELS=$PWD/external/models HOST_DATA=$PWD/external/data \
HOST_RESULTS=$PWD/external/results BENCH=skyrl EVAL_BENCH=swebench \
SCAFFOLD=openhands ALGO=sft \
  bash rl_engine/examples/shared/docker_run_train.sh
```

Run an in-house SWE training recipe (self-contained; no external SGLang):

```bash
WANDB_KEY=<key> bash scripts/train/swe/grpo/grpo_4b.sh
```

Run in-house SWE-agent eval (self-contained; no external SGLang):

```bash
HOST_CACHE=$PWD/external/cache DATASET=django_100 GPUS=0,1,2,3 \
  bash rl_engine/examples/swe_agent/scripts/docker_launch.sh swe-agent
```

---
> Source: [STACX/stacx](https://github.com/STACX/stacx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
