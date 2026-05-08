## gpu-ext

> - **NEVER use system pip.** Always use `uv` for Python dependency management.

# gpu_ext Repository Rules

## Hard Requirements

### Python Environment

- **NEVER use system pip.** Always use `uv` for Python dependency management.
- Each workload under `workloads/` has its own independent `.venv` managed by `uv`.
- Each workload has a `pyproject.toml` (declares deps) and `uv.lock` (version lock). Both must be committed.
- To run any Python script in a workload, use `uv run --directory <workload_dir>` or `cd <workload_dir> && uv run`.
- To initialize a workload: `cd workloads/<name> && uv sync`.
- Never install packages globally or with `pip install` outside a venv.
- Never use `--break-system-packages`.

### Workload Dependency Management

Each workload has independent version management:

| Workload | `pyproject.toml` deps | Source-installed packages |
|----------|----------------------|--------------------------|
| `llama.cpp` | requests, tqdm, huggingface-hub, numpy, matplotlib, pandas, psutil | — |
| `pytorch` | torch, psutil, numpy, matplotlib, torch-geometric | — |
| `faiss` | numpy<2, matplotlib | faiss (from local submodule build: `uv pip install -e faiss/build/faiss/python/`) |
| `vllm` | numpy, matplotlib | vllm (from local submodule: `uv pip install -e vllm/`) |

**Rule: 凡是本地有 source 的包，一律用本地 source 安装（`uv pip install -e <local_path>`），不从 PyPI 拉。** Source-installed packages 不写在 `pyproject.toml` 的 `dependencies` 里 — 它们需要先从源码构建，再 `uv pip install -e <path>` 装进 workload 的 `.venv`。

Known local sources:
- vLLM: `workloads/vllm/vllm/` (submodule, 带 UVM 支持的 fork)
- FAISS: `workloads/faiss/faiss/` (submodule, 需先 cmake build)
- llama.cpp: `workloads/llama.cpp/llama.cpp/` (submodule, 编译出 C++ binary，不是 Python 包)

### Workload Structure

Each workload directory (`workloads/<name>/`) must contain:

- `pyproject.toml` — declares Python dependencies for `uv`
- `uv.lock` — version lock file (committed to git)
- `.venv/` — uv-managed virtual environment (gitignored, recreated via `uv sync`)
- `run_exp*.sh` — one-click experiment scripts that save full logs and output results
- `results/` — benchmark output directory

### Experiment Scripts

- One script per experiment/figure from the paper (e.g., `run_exp1_expert_offload.sh` → Figure 6).
- Each script must:
  1. Check prerequisites (binaries, models, data)
  2. Run `python cleanup_gpu.py` before each benchmark config
  3. Save full logs to `results/exp<N>_<name>/<timestamp>/`
  4. Print a summary table at the end with paper reference values
- Scripts use `uv run` to ensure the correct venv is used.

### Model Downloads

- llama.cpp models: use `llama-server --gpt-oss-120b-default` or `--gpt-oss-20b-default` (auto-downloads from HuggingFace). Script: `workloads/llama.cpp/download_models.sh`.
- vLLM models: auto-downloaded by vLLM on first run.
- SIFT dataset: `bash workloads/faiss/download_sift.sh`.
- **Never use `huggingface-cli` directly** — use llama.cpp's built-in download flags.
- Download scripts must be idempotent (skip if files already exist).

### GPU Cleanup

Always run `python workloads/cleanup_gpu.py` before benchmarks to kill stale GPU processes.

### Experiment Execution

- **Experiments MUST run serially, NEVER in parallel.** BPF struct_ops is a global singleton — only one can be loaded at a time. The GPU is also a shared resource. Two concurrent experiments will corrupt each other's results or crash.
- When using subagents: multiple subagents may write code and build in parallel, but only ONE subagent may run a GPU/BPF experiment at any given time. The next experiment starts only after the previous one finishes and cleans up.
- Always kill BPF processes and run `cleanup_struct_ops_tool` between experiment configs.

### Git

- Large files (models, datasets, `.venv/`, build artifacts) are gitignored.
- Never commit `.venv/`, `build/`, `*.gguf`, `*.bvecs`, or `__pycache__/`.
- DO commit `pyproject.toml` and `uv.lock` for every workload.
- **DO commit benchmark result JSON files** (`results/*.json`, `result/*.json`). They are small (4K each) and serve as experiment records. Always include them when committing after running experiments.

### Codex CLI as Subagent

OpenAI Codex CLI is available on this machine (`codex-cli 0.111.0`, model `gpt-5.4`). Use it for **independent code-writing tasks**.

```bash
# Non-interactive execution — no sandbox, no prompts
codex exec --dangerously-bypass-approvals-and-sandbox "your prompt here"

# With a specific working directory
codex exec --dangerously-bypass-approvals-and-sandbox -C /path/to/dir "your prompt here"
```

**When to use Codex / subagent (Agent tool)**:
- Complex, independent coding tasks (implement a new BPF program, modify a benchmark, etc.)
- Tasks that don't need intermediate decisions from the main conversation
- Parallelizable work: multiple subagents can write code simultaneously (but experiments must still run serially)

**When NOT to use**:
- Tasks requiring GPU/BPF execution (need sudo, struct_ops singleton)
- Tasks requiring interactive decisions or context from the main conversation

### Task Delegation Principles

- **Complex independent tasks → subagent or codex**. Don't do everything in the main conversation.
- **Research/investigation → Agent tool** (subagent_type=Explore or general-purpose)
- **Code writing → codex exec** or Agent tool (general-purpose)
- **Experiments → Agent tool** (must be serial, one at a time)
- **Always give subagents/codex complete context**: file paths, function signatures, constraints, expected output format.

### Paper Writing

- When writing or reviewing paper text (LaTeX), follow the paper-writing skill at `.claude/skills/paper-writer.md`.
- This includes hard rules from co-author (arq) reviews, anti-patterns identified during resubmission, and self-review questions.
- Key rules: no em-dashes, no "our" for prior work (double-blind), one idea per paragraph, evidence before explanation.

### Paper Reproduction

All experiments must match the paper's configuration (models, dataset sizes, number of trials). Software versions use whatever is locally installed/built from source — do not pin to specific versions.

All benchmark results: 10 trials, geometric mean (unless noted otherwise).

---
> Source: [eunomia-bpf/gpu_ext](https://github.com/eunomia-bpf/gpu_ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
