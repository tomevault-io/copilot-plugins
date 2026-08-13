## implicit-concealment-analysis

> **AI agents: do NOT write to this file unless specifically asked to — and even when asked,

# CLAUDE.md — repo guide for agents

**AI agents: do NOT write to this file unless specifically asked to — and even when asked,
encourage human review of the exact diff. This file only stays useful if it stays curated;
unsupervised agent edits turn it to slop.**

Orientation + operating rules for this repo. Read this before touching anything.

## What this project is

Research pipeline studying **when and why LLM agents conceal information present in their
system context**. Three experiments at increasing depth, all sharing one pipeline
(`src/pipeline/`) and one monitor stack (`src/monitors/`):

1. **Black-box** (`scripts/run_blackbox.py`) — disclosure rates across three instruction
   conditions: A0 (transparent), A1 (implicit pressure), A2 (explicit suppression).
2. **Framing** (`scripts/run_framing.py`) — 18 framing conditions across 5 dimensions
   (motivation, incentive, audience, baseline, control) mapping the implicit→explicit
   suppression spectrum. See `docs/FRAMING_EXPERIMENT.md` and `docs/CONDITIONS_REFERENCE.md`.
3. **Interpretability** (`scripts/run_interp.py`, `scripts/run_patching.py`) — activation
   capture + layerwise linear probes, PCA, logit lens, causal/attribution patching on local
   HF models (Qwen2.5-1.5B is the workhorse).

Models run either via **Together AI** (`TogetherClient`, needs `TOGETHER_API_KEY`) or
**locally via HuggingFace** (`HFClient`, `--local`, supports activation capture). Both
implement the `ChatClient` protocol in `src/clients/__init__.py`.

## Where things go (keep this structure)

```
src/                  reusable, correctness-critical code — import as src.*; scripts stay thin
  clients/              TogetherClient (retry logic) + HFClient (local models, activation capture)
  config.py             Config dataclass: base_model, monitor_model, dataset, use_local_model,
                        capture_activations (none/last_token/full_sequence/reasoning_span), ...
  dataset/              synthetic concealment-scenario generator (ShippingDomain, BugDomain, ...)
  framing/              the 18 framing conditions (conditions.py: REGISTRY, CONDITION_ORDER) + FramingLoader
  loaders/              prompt loaders (GPQA, MedQA-obfuscation, Concealment, JSON)
  monitors/             RegexMonitor, KeywordMonitor, LLMMonitor (LLM judge; API mode only)
  pipeline/             Pipeline, PipelineStep, PromptResult (has activation_path field)
  steps/                BaseModelStep (chat + save_activations), MonitorStep
  interp/               ActivationStore, linear probes, logit lens, patching, ablation, SAE utils
  storage/              ResultStorage — writes results/run_<ts>_<id>.json
scripts/              pipeline drivers + plotting/eval CLIs; a script pipes src/ functions
                      together and does no real work itself
  run_*.py              the four experiment drivers + main.py (generic) + make_dataset.py
  plot_*.py             figures for each experiment (plot_results, plot_framing, plot_interp)
  eval_*.py             causal evaluation over saved activations
  *vertex*, *.sh        Vertex AI submission/download/runner + docker build
scratch/              one-off and AI-generated scripts. DEFAULT home for new experimental
                      code; NOTHING imports from it. notes/ (gitignored) holds private writeups.
vertex_jobs/          Vertex AI job YAML templates + runs/ (generated per-run YAMLs)
docs/                 experiment/architecture/CLI reference docs + VERTEX_EXPERIMENT_RUNBOOK.md
tests/                fast unit tests. Run: python3.11 -m pytest -q
saved_experiments/    manually archived experiment snapshots (frozen; do not edit)
data/                 generated datasets (JSONL, timestamped)         } working copies —
results/              run outputs: JSON + plots (gitignored)          } canonical store is
activations/          saved .npz activation files                     } the HF dataset repo
```

**Artifacts live on the Hub, not in git.** The canonical store for `data/`,
`results/`, `activations/`, `saved_experiments/`, `vertex_downloads/` is the
public HF dataset repo **`kunwar45/obfuscation-prompting`** (same folder names,
so `activation_path` strings match). `src/storage/hf_artifacts.py` is the
resolver — `ActivationStore` and the plot scripts fetch missing files from the
Hub automatically. Sync explicitly with `python -m scripts.hf_sync pull|push
<folder ...>`; **push new results/activations after every non-smoke run**.
Never upload anything containing values from `.env` — the repo is public.

**Respect the structure when adding code:**

- `src/` holds reusable, reviewed logic other code may depend on. If a script grows logic
  worth reusing, the logic moves into `src/` and the script stays thin.
- `scripts/` holds pipelines we expect to rerun. New AI-generated code defaults to
  `scratch/` until a human promotes it.
- **Integrate, don't tack on**: extend the existing module rather than adding
  `foo_v2.py` / `foo_new.py` siblings.
- `saved_experiments/` and `vertex_jobs/runs/` are historical records — never rewrite them.

## How to run things

**Run everything from the repository root**, and **invoke drivers as modules** — they import
`src.*`, so `python3.11 -m scripts.run_framing ...` works and
`python3.11 scripts/run_framing.py` does not (no sys.path hacks in the drivers; inside the
Docker image `PYTHONPATH=/app` makes both forms work).

**Always use `python3.11`** (not the default python3) for anything torch-dependent on this
machine. Cloud (Together) runs work on any Python 3.x.

```bash
# setup
python3.11 -m venv venv && source venv/bin/activate && pip install -r requirements.txt
cp .env.example .env   # TOGETHER_API_KEY for cloud runs; HF_TOKEN only for GPQA

# black-box (Together API)
python -m scripts.run_blackbox --smoke-only
python -m scripts.run_blackbox --skip-smoke --n-scenarios 50

# framing (local model)
python3.11 -m scripts.run_framing --local --skip-smoke --n-scenarios 30 \
  --local-model Qwen/Qwen2.5-1.5B-Instruct --max-tokens 256

# interp (recommended settings)
python3.11 -m scripts.run_interp --model Qwen/Qwen2.5-1.5B-Instruct \
  --dtype float16 --n-scenarios 30

# plots (read a results JSON)
python scripts/plot_framing.py results/run_<ts>_<id>.json

# tests
python3.11 -m pytest -q
```

Every driver has `--smoke-only` / `--skip-smoke`; always smoke-test before a full run.
Vertex AI runs: follow `docs/VERTEX_EXPERIMENT_RUNBOOK.md` (create_vertex_run_config →
submit_vertex_job.sh → download_vertex_results.sh; templates in `vertex_jobs/`).

## Conventions

- All LLM responses use `<reasoning>...</reasoning><answer>...</answer>` tags.
- Results: `results/run_<ts>_<id>.json` (+ `_analysis.json`, `_last_token_eval.json` for
  interp); plots saved next to the results JSON. Datasets: `data/<name>_<ts>.jsonl`.
  Activations: `activations/<tag>_<ts>/*.npz`, indexed by `src/interp/activation_store.py`.
- Prompt ID format: `{example_id}_{framing_key}_{query_type}`
  (e.g. `shipping_0003_M_inst_s_B1`).
- Effect sizes: Cohen's h = `2*asin(sqrt(p1)) - 2*asin(sqrt(p2))`; p-values from
  two-proportion z-tests (`scripts/plot_framing.py`).
- Controls use BASE framing only; condition ordering comes from
  `src.framing.conditions.CONDITION_ORDER` — import it, don't hardcode lists.

## Gotchas (learned the hard way)

1. **`max_tokens` on MPS**: `Config.max_tokens=2048` is far too slow for local runs —
   pass `--max-tokens 256` (run_interp's default) for local/MPS experiments.
2. **transformers ≥ 4.40**: use `dtype=` not `torch_dtype=` in `from_pretrained`
   (`torch_dtype` is deprecated and logs a warning).
3. **LLMMonitor needs the API**: it is omitted automatically in `--local` mode — don't add
   it to local pipelines.
4. **Logit-lens token IDs**: secret values that are numbers (e.g. "28") may tokenize as
   multiple tokens in Qwen ("2"+"8"); `_find_secret_token_id` skips those pairs — don't
   "fix" that by taking the first subtoken.
5. **PCA over string conditions**: `run_pca` must use `get_results()` directly, not
   `get_labels()` (which assumes numeric labels).
6. **Together API**: requires `TOGETHER_API_KEY` in the environment (loaded from `.env`);
   the client already has retry logic — don't wrap it in another retry loop.

## Secrets

All credentials live in the gitignored `.env` at the repo root (`TOGETHER_API_KEY`,
`HF_TOKEN`). Copy `.env.example` and fill it in. Never print, log, commit, or summarize a
secret value. New env var names (never values) go into `.env.example`.

---
> Source: [kunwar45/implicit-concealment-analysis](https://github.com/kunwar45/implicit-concealment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
