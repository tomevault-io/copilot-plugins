## collider-bench

> You are a coding agent setting up **Collider-Bench** for a researcher. The

# CLAUDE.md

You are a coding agent setting up **Collider-Bench** for a researcher. The
human will not run any of the commands below — you will. Read this top-to-
bottom, execute, and only ask the human when you hit a decision the file
doesn't cover.

## What you're setting up

A benchmark that scores LLM-agent recasts of CMS LHC analyses. Each task
gives an agent a published paper + a null-filled HEPData-style YAML; the
agent has to build a simulation + selection pipeline and fill predicted
bin yields. The harness scores the filled YAML against a hidden reference
and writes per-run artifacts under `runs/`.

**The HEP stack (MadGraph5, Pythia8, Delphes, Prospino) lives only inside
the container image** at `ghcr.io/dfaroughy/lhc-bench:latest`. The host
needs none of it — just Python + a container runtime + one agent CLI.

## Setup sequence (execute in order)

```bash
# 1. Venv + harness — no conda required
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# 2. Container engine — pick one that's available on this host
#    Mac:   brew install podman && podman machine init && podman machine start
#    Linux: docker / podman / apptainer are typically pre-installed
docker pull ghcr.io/dfaroughy/lhc-bench:latest
# (or: podman pull / apptainer pull docker://...)

# 3. One vendor agent CLI on the host (the sandbox bind-mounts it in)
npm i -g @anthropic-ai/claude-code   # claude → ~/.local/bin/claude
# alternatives: @openai/codex, @google/gemini-cli, aider, forge-code

# 4. Pick an auth mode (see "Auth modes" below) and configure it
#    — for `auth: api` (configs/claude.yaml default):
export ANTHROPIC_API_KEY=...      # claude
# OPENAI_API_KEY=, GEMINI_API_KEY=, DEEPSEEK_API_KEY= for other runners
#    — for `auth: oauth` (cheaper / subscription-funded):
# claude /login        # (or: codex /login, gemini auth login)
# Then change configs/claude.yaml's `auth: api` → `auth: oauth`.

# 5. Smoke test — should print run paths and exit 0 within ~20 min
scripts/run-agent --config configs/claude.yaml --task sus-16-046_sim-T5Wg
```

The smoke test runs the smallest task (`sus-16-046_sim-T5Wg`, ~4 bins) end
to end and writes a scored run under `runs/claude_opus-4-7/...`. If it
fails, see the pitfall table below before asking the human.

## Auth modes — `oauth` vs `api`

Each major vendor (Anthropic / OpenAI / Google) supports two paths. **Pick
the cheaper one (`oauth`) unless the user explicitly says otherwise.**

| Mode | What it uses | When to pick it | Caveats |
|---|---|---|---|
| `oauth` | The vendor CLI's own login session (`claude /login`, `codex /login`, `gemini auth login`) — usage gets billed to that account's subscription / free quota. | **Default for individual users.** A Claude / ChatGPT / Gemini subscription is ~$20–200/month flat and covers thousands of runs. Most researchers already have one. | Needs an interactive login on the host first. Refresh tokens rotate — the harness syncs them back to the host on exit ([sandbox.py:_sync_credentials_back_to_host](agent_runtime/sandbox.py)). Batch / headless / SLURM execution is fragile. |
| `api` | A direct API key (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `DEEPSEEK_API_KEY`). | Production batch eval, SLURM jobs, CI, parallel sweeps, anywhere you need headless reliability.| No interactive login, just an exported env var. The validator ([config.py:validate_api_auth_env](agent_runtime/config.py)) fails fast if the key is missing, so misconfigured runs don't spend tokens. |

In configs:

```yaml
auth: oauth     # uses the existing CLI login on the host
# OR
auth: api       # requires the matching *_API_KEY env var
```

Both shipped configs (`configs/claude.yaml`, `configs/claude_slurm.yaml`)
default to `auth: api` for batch / SLURM convenience. **If you're setting
up for a single human researcher, switch the field to `oauth` and have them
run `claude /login` once.** That alone can reduce their per-run cost from
$10 to a few cents (subscription-amortized).

## Sanity checks (before the smoke test)

```bash
# Harness imports cleanly
python -c "import agent_runtime.config, agent_runtime.sandbox; print('OK')"

# Container engine is reachable
docker info >/dev/null && echo "docker OK"

# Image is local
docker images | grep lhc-bench

# Agent CLI is on PATH
command -v claude && claude --version

# Test suite (no SLURM, no LLM, ~100s)
python -m pytest -q
```

All five should pass before invoking `scripts/run-agent`.

## Repository layout

Three top-level layers — read code only when something below isn't enough.

| Layer | Path | What's in it |
|---|---|---|
| Benchmark | [`ColliderBench/`](ColliderBench/) | Tasks ([`tasks/`](ColliderBench/tasks/)), scorer ([`Evals/`](ColliderBench/Evals/)), agent-facing CLIs ([`tools/CLI/`](ColliderBench/tools/CLI/)), CLI shims ([`bin/`](ColliderBench/bin/)) |
| Harness | [`agent_runtime/`](agent_runtime/) | Entrypoint ([`launch.py`](agent_runtime/launch.py)), YAML loader+validator ([`config.py`](agent_runtime/config.py)), sandbox abstraction ([`sandbox.py`](agent_runtime/sandbox.py)), runner registry ([`vendors.py`](agent_runtime/vendors.py)), shell glue ([`shell/agent_env.sh`](agent_runtime/shell/agent_env.sh)) |
| Agents | [`agents/`](agents/) | Only public agent: [`simple/`](agents/simple/) (one LLM call, no retries, no planning) |

Configs: only two shipped — [`configs/claude.yaml`](configs/claude.yaml)
(local) and [`configs/claude_slurm.yaml`](configs/claude_slurm.yaml)
(Perlmutter SLURM). Full schema in [`configs/CONFIG.md`](configs/CONFIG.md).

## Output of one run

```
runs/<runner>_<model>/<run-label>/<task>_<hash>/
├── workspace/         agent's filesystem (rw inside sandbox)
│   ├── results/       null-filled YAMLs — agent fills these in place
│   ├── bin/           run-analysis + agent CLI on $PATH
│   └── papers/        symlink to the paper PDF
├── eval/
│   ├── score.json     primary metric output (relative L²)
│   ├── plots/         reference vs agent histograms
│   └── judge_*.{json,md}   only if --judge was passed
├── run_info.json      wall_s, cost_usd, tokens
└── *.stream.jsonl     raw agent transcript
```

`score.json` key fields: `task_id`, `n_bins`, `n_filled`,
`normalization.{Delta, relative_l2}`, `shape.{relative_l2, jensen_shannon}`.

## Constraints (do NOT do these)

- **Do not** create or modify anything under `ColliderBench/tasks/shared/*/reference/` — that's the hidden answer key. It isn't mounted into the agent sandbox; if you can read it from the host you're outside the sandbox boundary.
- **Do not** run `python3 analysis.py` directly inside an agent workspace — use `bin/run-analysis`, which sets `PYTHONPATH` and gates with `agent_runtime/preflight.py`.
- **Do not** call container engines directly (`docker run …`) for scored runs — `scripts/run-agent` is the only sanctioned entrypoint. It builds the sandboxed workspace, hides the reference, and writes provenance to `run_info.json`.
- **Do not** install conda, MadGraph, Pythia8, Delphes, or any HEP library on the host. They live in the image and the host doesn't need them. If a tool seems to require them, you're invoking a host-side path by mistake.
- **Do not** modify `pyproject.toml` to add scientific-Python deps unless you can prove the harness *imports* them on the host (the image holds the rest).

## Common pitfalls

| Symptom | Cause + remedy |
|---|---|
| `Task missing: sus-16-XXX_shape-YYY` | Active corpus is sim-only. Shape/yield variants are private and not shipped. Use a `_sim-` task id. |
| `Missing conda environment 'lhc_analysis'` from `agent_env.sh` | You're running outside a venv with the harness deps. Re-activate the venv and re-check `python -c "import yaml,pydantic,numpy,matplotlib,mplhep"`. |
| Agent CLI not found inside container | The host-side CLI isn't on `$PATH`. Run `which claude` (or `codex`/`gemini`) and confirm it's under `$HOME` — the sandbox bind-mounts that path. |
| `ANTHROPIC_API_KEY` missing | When `auth: api` is set in the config, the harness fails fast at load time. Either export the key or switch the config to `auth: oauth` (then run `claude /login` once on the host). |
| `ColliderBench/tools/sim/` doesn't exist | Expected — that tree is gitignored. The image's `/opt/sim/` is what the agent uses. Don't try to recreate the host tree. |
| Run hangs after Pythia | Known Pythia 8.312 bug with `info.atEndOfFile()`. The fix pattern is in [`SIMULATE.md`](ColliderBench/tools/CLI/SIMULATE.md) — count LHE events upfront, iterate exactly N times. |
| Mac arm64 slowness | The image is amd64 only. Apple Silicon runs it under Rosetta — slow, occasionally flaky. Prefer a Linux box or cloud VM for real runs. |

## Where to look up details

| Topic | File |
|---|---|
| Config schema, validation, runtime path | [`configs/CONFIG.md`](configs/CONFIG.md) |
| Sandbox backends + contract | [`agent_runtime/SANDBOX.md`](agent_runtime/SANDBOX.md) |
| Image contents + rebuild | [`docker/README.md`](docker/README.md), [`docker/Dockerfile`](docker/Dockerfile) |
| What tools the agent sees inside the sandbox | [`ColliderBench/tools/TOOLS.md`](ColliderBench/tools/TOOLS.md), [`ColliderBench/tools/CLI/*.md`](ColliderBench/tools/CLI/) |
| Scoring math | [`ColliderBench/Evals/score.py`](ColliderBench/Evals/score.py), [`ColliderBench/Evals/metrics/`](ColliderBench/Evals/metrics/) |
| Judge prompt (provenance audit) | [`ColliderBench/Evals/judge_rubric.md`](ColliderBench/Evals/judge_rubric.md) |
| One small task as exemplar | [`ColliderBench/tasks/sus-16-046_sim-T5Wg/`](ColliderBench/tasks/sus-16-046_sim-T5Wg/) |

## Test + lint commands

```bash
python -m pytest -q                  # full offline smoke (~100s)
python -m pytest -x -v               # stop on first fail, verbose
ruff check . && ruff format .        # lint + format (matches pre-commit)
```

Python 3.10+ is required (`from __future__ import annotations` everywhere,
plus `match` statements in a few places).

---
> Source: [dfaroughy/Collider-Bench](https://github.com/dfaroughy/Collider-Bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
