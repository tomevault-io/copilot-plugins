## its-openshift-mcp

> Benchmarking platform for evaluating how well an AI agent troubleshoots Kubernetes/OpenShift infrastructure problems using a local OpenShift cluster, the Lightspeed service, and troubleshooting demo scenarios.

# OLS Evals Project

Benchmarking platform for evaluating how well an AI agent troubleshoots Kubernetes/OpenShift infrastructure problems using a local OpenShift cluster, the Lightspeed service, and troubleshooting demo scenarios.

## Project Layout

```
mcp/
├── CLAUDE.md                     # This file
├── README.md                     # Project overview, architecture, findings
├── eval/                         # Evaluation framework config and scripts
│   ├── eval.yaml                 # Scenario definitions (queries, contexts, expected responses)
│   ├── system.yaml               # Framework config: judge LLM, metrics, rubrics
│   ├── system_gpt5mini.yaml      # Config variant: gpt-5-mini agent
│   ├── system_gpt41mini.yaml     # Config variant: gpt-4.1-mini agent
│   ├── system_prompt.txt         # Reusable system prompt for experiments
│   └── scripts/
│       ├── average_evaluation/   # Multi-iteration runner (run_iterations.py)
│       └── dashboard/            # Visualization dashboard (docker.sh / podman.sh)
├── scenarios/                    # Troubleshooting scenarios (K8s manifests + deploy scripts)
│   ├── oom/                      # OOM kill — memory-leaking pod (eval tag: `oom`)
│   ├── networkpolicy/            # NetworkPolicy label mismatch (eval tag: `networkpolicy`)
│   └── payments/                 # Cross-namespace DB exhaustion + red herring (eval tag: `demo_b`)
├── algorithms/                   # Inference-time algorithms (canonical implementations)
│   └── inference_scaling.py      # its_hub integration for inference-time scaling
├── patches/                      # Patches to apply to upstream repos
│   └── inference_scaling.patch   # Lightspeed service integration for inference-time scaling
├── scripts/                      # Automation
│   ├── setup.sh                  # Full setup: clone repos, install deps, apply patches, deploy
│   └── run_eval.sh               # Controlled eval runner: redeploys scenarios between iterations
├── mcp_config.toml               # MCP server tool access config
├── docs/                         # Documentation
│   ├── SETUP.md                  # Manual setup guide (for when setup.sh breaks)
│   ├── SELF_CONSISTENCY.md       # Algorithm design and results
│   ├── TOOL.md                   # Tool calling architecture (agentic loop, MCP, token budget)
│   └── DEMOS.md                  # All 8 demo scenarios with expected tool calls
├── _benchmark/                   # Runtime artifacts (gitignored, created by setup.sh)
│   ├── lightspeed-service/       # Cloned + patched upstream agent (Python venv with lightspeed-eval)
│   ├── openshift-mcp-server/     # Cloned + built MCP server (Go binary)
│   ├── its_hub/                  # Cloned its_hub (inference-time scaling algorithms)
│   └── eval_output/              # Raw eval output (CSV, JSON)
├── _results/                     # Dated results writeups (gitignored)
└── _traces/                      # Curated trace data by config (gitignored)
```

## Components

**[OpenShift Local (CRC)](https://console.redhat.com/openshift/create/local)** — a local single-node OpenShift cluster in a VM. The scenarios (`scenarios/`) deploy intentionally broken workloads here.

**[openshift-mcp-server](https://github.com/openshift/openshift-mcp-server)** — the **hands**. A Go binary that exposes cluster operations (list pods, read logs, get events, describe resources) as structured tools via the Model Context Protocol. No AI — just wraps the Kubernetes API. Runs on port 8085.

**[lightspeed-service](https://github.com/openshift/lightspeed-service)** — the **brain**. A Python/FastAPI service that takes a user question, sends it to an LLM (e.g. `gpt-5-mini`), and lets the LLM decide which MCP tools to call. Runs the agentic loop: prompt → LLM → tool calls → execute via MCP server → append results → repeat until the LLM has enough evidence to answer. The inference-time scaling integration (`algorithms/inference_scaling.py`) plugs in here, using its_hub to vote on which tool call to make at each round. Runs on port 8080.

**[lightspeed-evaluation](https://github.com/lightspeed-core/lightspeed-evaluation)** (v0.4.0) — the **referee**. Provides the `lightspeed-eval` CLI and metric frameworks (GEval, DeepEval, RAGAS, custom). Installed as a Python dependency of `lightspeed-service` via `uv sync` — the CLI lives at `_benchmark/lightspeed-service/.venv/bin/lightspeed-eval`. Reads `eval/eval.yaml` (scenarios) and `eval/system.yaml` (metrics/rubrics).

**Judge LLM (gpt-4.1)** — a stronger model that scores agent responses. Checks whether the agent found real cluster evidence or gave generic suggestions, using criteria defined in `eval/system.yaml`.

## Architecture

```
eval/eval.yaml ──► Lightspeed API (localhost:8080) ──► MCP tools ──► OpenShift cluster (CRC)
                         │ responses                       ▲
                         ▼                        openshift-mcp-server (localhost:8085/mcp)
                   Judge LLM (gpt-4.1)
                         │
                         ▼
                   scores ──► CSV/JSON in _benchmark/eval_output/
```

See `docs/TOOL.md` for the full agentic loop (tool discovery, LangChain binding, iteration, token budget).

## Upstream Sources

**Cloned at setup time** into `_benchmark/`:
- [openshift/lightspeed-service](https://github.com/openshift/lightspeed-service) — inference-time scaling patch applied on top
- [openshift/openshift-mcp-server](https://github.com/openshift/openshift-mcp-server) — used as-is
- [Red-Hat-AI-Innovation-Team/its_hub](https://github.com/Red-Hat-AI-Innovation-Team/its_hub) (branch `feature/tool-flat-all`) — inference-time scaling algorithms, installed editably into lightspeed-service venv

**Installed as Python dependency** (by `uv sync`):
- [lightspeed-core/lightspeed-evaluation](https://github.com/lightspeed-core/lightspeed-evaluation) (v0.4.0) — `lightspeed-eval` CLI + metric frameworks

**Forked into this repo** from internal GitLab:
- [criolo/ols-evals](https://gitlab.cee.redhat.com/criolo/ols-evals) (`experiments/its`) → `eval/` — track upstream for metric implementation changes and new CLI features
- [afalossi/troubleshooting-demos](https://gitlab.cee.redhat.com/afalossi/troubleshooting-demos) (`experiments/its`) → `scenarios/` — track upstream for new demo scenarios (demo1-5 exist but aren't wired into evals yet)

## Setup

The user may ask you to set up the benchmark environment. Here's how to help:

### Prerequisites check

Before running setup, verify the user has:
1. **CRC running** — `oc whoami` should return `kubeadmin`. If not, guide them through `crc start` and `oc login`.
2. **Go installed** — `go version`
3. **Python 3.11+ and uv** — `python3 --version` and `uv --version`
4. **OpenAI API key** — `openai_key.txt` must exist in the repo root. Ask the user to create it if missing.

### Automated setup

```bash
./scripts/setup.sh
```

This script:
1. Clones [openshift/lightspeed-service](https://github.com/openshift/lightspeed-service) into `_benchmark/lightspeed-service/` — the AI agent under test
2. Clones [openshift/openshift-mcp-server](https://github.com/openshift/openshift-mcp-server) into `_benchmark/openshift-mcp-server/` — the cluster tool provider
3. Runs `uv sync` in lightspeed-service (creates Python venv, installs [lightspeed-core/lightspeed-evaluation](https://github.com/lightspeed-core/lightspeed-evaluation) which provides the `lightspeed-eval` CLI)
4. Builds the MCP server Go binary
5. Applies the inference-time scaling patch (`patches/inference_scaling.patch`)
6. Clones [its_hub](https://github.com/Red-Hat-AI-Innovation-Team/its_hub) (branch `feature/tool-flat-all`) into `_benchmark/its_hub/` and installs it editably into the lightspeed-service venv
7. Generates `olsconfig.yaml` with the user's OpenAI key path
8. Deploys all scenarios to the cluster

The script is idempotent — it skips steps already done.

Use `./scripts/setup.sh --skip-cluster` to skip scenario deployment if the cluster isn't running.

### After setup

The user needs to start two long-running services in separate terminals before running evals. You can help by:
1. Telling them the exact commands (see "Running evaluations" below)
2. Running `scripts/run_eval.sh` which starts lightspeed-service in the background automatically
3. Verifying the stack is healthy: `curl -s http://localhost:8080/readiness` (Lightspeed) and `curl -s http://localhost:8085/mcp` (MCP server)

### Troubleshooting common issues

- **`lightspeed-eval` not found**: the venv shebang may be stale if `_benchmark/lightspeed-service` was moved. Fix: `cd _benchmark/lightspeed-service && uv sync`
- **Tools not loading**: check that `olsconfig.yaml` has `url: http://localhost:8085/mcp` (with `/mcp` suffix). Without it, tools silently fail to load.
- **OOM scenario not working**: `OOMKilled` events age out after ~1h. Redeploy: `cd scenarios/oom && make deploy`
- **Payments scenario stale**: leak signal fades ~5 min after `make break`. Redeploy: `cd scenarios/payments && make deploy && make break && make break-redherring`

See `docs/SETUP.md` for the manual setup steps (for when `setup.sh` breaks).

### Running evaluations

```bash
# Start MCP server (terminal 1)
cd _benchmark/openshift-mcp-server
./openshift-mcp-server --config ../../mcp_config.toml --port 8085

# Start Lightspeed service (terminal 2)
cd _benchmark/lightspeed-service
make run

# Single eval run (terminal 3)
cd eval
export OPENAI_API_KEY=$(cat ../openai_key.txt)
export PATH="../_benchmark/lightspeed-service/.venv/bin:$PATH"
API_KEY=$(oc whoami -t) lightspeed-eval \
  --system-config system_gpt5mini.yaml \
  --eval-data eval.yaml \
  --output-dir ../_benchmark/eval_output

# Controlled iterations (redeploys scenarios between each run)
./scripts/run_eval.sh 4 _benchmark/eval_output 3
```

### View results dashboard

```bash
cd eval
./scripts/dashboard/podman.sh ../_benchmark/eval_output
# Visit http://localhost:8888/
```

## Evaluation Scenarios

| Tag              | Scenario                              | Turns | Namespace(s) |
|------------------|---------------------------------------|-------|--------------|
| `oom`            | OOM kill investigation (memory leak)  | 1     | `demo-oom` |
| `networkpolicy`  | NetworkPolicy label mismatch          | 3     | `demo-netpol` |
| `demo_b`         | Payments failure / PostgreSQL connection exhaustion | 3 | `payments`, `shared-services` |

See `docs/DEMOS.md` for all 8 demo scenarios and expected tool calls.

## Key Config Files

- **`eval/system.yaml`**: Judge LLM (gpt-4.1), metrics/rubrics, thresholds
- **`eval/system_gpt5mini.yaml`**: Same but targets gpt-5-mini as agent
- **`eval/eval.yaml`**: Scenario definitions (queries, contexts, expected responses, metrics)
- **`_benchmark/lightspeed-service/olsconfig.yaml`**: Lightspeed config (provider, models, MCP server URL, auth)
- **`mcp_config.toml`**: MCP server tool access controls

## MCP Server

The `/mcp` suffix in the URL is **required** — without it, the connection silently fails and tools are never loaded (the agent falls back to generic answers).

```yaml
# In olsconfig.yaml:
mcp_servers:
  - name: openshift
    url: http://localhost:8085/mcp    # MUST include /mcp path
```

Available tools: `pods_list`, `pods_list_in_namespace`, `pods_get`, `pods_log`, `pods_top`, `pods_exec`, `events_list`, `namespaces_list`, `resources_get`, `resources_list`, `resources_create_or_update`, and more (20 total).

## Metrics

**Discriminating** (real signal):
- `geval:generic_troubleshooting_experience` — penalizes generic suggestions without cluster evidence
- `geval:k8s_oom_root_cause_analysis` — OOM-specific: exit code 137, memory limits, leak source
- `geval:payments_troubleshooting_experience` — cross-namespace awareness, connection exhaustion, rollout correlation
- `geval:troubleshooting_continuity` — multi-turn stateful debugging

**Supporting**: `custom:answer_correctness`, `deepeval:conversation_completeness`, `deepeval:conversation_relevancy`, `deepeval:knowledge_retention`

All metrics score 0.0–1.0 with a 0.7 pass threshold.

## Current Results

Best result: **gpt-5-mini with self-consistency N=8 — 83.3% pass rate** (20 Pass / 4 Fail).

| Configuration | Pass Rate |
|---------------|-----------|
| gpt-4.1-mini, no tools | 16.7% |
| gpt-5-mini, no tools | 45.8% |
| gpt-5-mini, working tools | 58.3% |
| gpt-5-mini, tools + SC N=8 | **83.3%** |
| gpt-4.1-mini, tools + SC N=8 | 33.3% |

See `_results/` for the full its_hub sweep results.

## Inference-Time Scaling

Canonical implementation: `algorithms/inference_scaling.py` (its_hub integration)
Integration patch: `patches/inference_scaling.patch`
Design doc: `docs/SELF_CONSISTENCY.md`

N parallel LLM calls per tool-calling round, majority vote on `(tool_name, frozen_args)` signatures. Amplifies correct reasoning (+25pp on gpt-5-mini) but locks in systematic errors (-4pp on gpt-4.1-mini). Implemented via its_hub which also supports BestOfN and PlanningWrapper algorithms.

## Cluster State Freshness

Scenarios must be redeployed before each eval iteration:
- **OOM**: events age out after ~1h
- **Payments**: leak signal fades after ~5 min post-`make break`
- **NetworkPolicy**: stable, no redeployment needed

`scripts/run_eval.sh` handles this automatically.

## Dependencies

- [OpenShift Local (CRC)](https://console.redhat.com/openshift/create/local) with `oc` CLI
- Python 3.11+ with [uv](https://docs.astral.sh/uv/)
- `lightspeed-eval` CLI (installed into `_benchmark/lightspeed-service/.venv/bin/` by `uv sync`)
- Go (to build openshift-mcp-server)
- OpenAI API key (for both agent model and judge LLM)
- Docker or Podman (for dashboard)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Red-Hat-AI-Innovation-Team)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Red-Hat-AI-Innovation-Team)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
