## quantlab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

A quantitative trading system that fuses three open-source projects into a unified T+1 A-share trading pipeline:

- **qlib/** — Qlib: Data infrastructure, Alpha158 factor engine, backtesting framework (Microsoft)
- **Kronos/** — Foundation Transformer model for K-line sequence forecasting (Microsoft)
- **RD-Agent/** — LLM-driven evolutionary R&D agent for automated factor/model discovery (Microsoft)
- **quantlab/** — Integration layer that wires the three projects into a daily trading pipeline

System design docs: `DESIGN.md` (architecture overview) and `DETAIL_DESIGN.md` (module specs M1–M9).

## quantlab (Integration Layer)

### Setup

```bash
pip install -e .                      # Install quantlab as editable package
pip install -e ./qlib                  # Install qlib (required dependency)
python quantlab/setup_and_verify.py   # 5-step env verification + Qlib data download

# Optional: set paths for Kronos / RD-Agent if not in sibling directories
# export QUANTLAB_KRONOS_PATH=/path/to/Kronos
# export QUANTLAB_RDAGENT_PATH=/path/to/RD-Agent
# export QUANTLAB_QLIB_DIR=/path/to/qlib    # only needed for dump_bin scripts
```

### Test

```bash
python -m pytest quantlab/tests/ -v                          # All tests
python -m pytest quantlab/tests/test_signal_alpha.py -v      # Single module
python -m pytest quantlab/tests/test_signal_alpha.py::test_name -v  # Single test
```

### Architecture

quantlab implements the pipeline from DETAIL_DESIGN.md. Code is organized into subdirectories by concern:

```
M1   data/data_manager.py          — Data update (Yahoo/Baostock/CSV → Qlib bin) + time-isolated access
M1.5 data/data_viewer.py           — CSV export, K-line charts (Plotly), portfolio overview
M2   signal/signal_alpha.py        — Alpha158 + LightGBM (daily trend/momentum signal)
M3   signal/signal_kronos.py       — Kronos 5-day forecast (K-line pattern signal)
M4   signal/signal_rdagent.py      — RD-Agent evolved factors (mean-reversion signal)
M5   signal/signal_ensemble.py     — Signal fusion (rank normalization, expanding IC weighting)
M6   execution/execution.py        — T+1 open-price trade execution
M7   risk_control/risk_control.py  — 3-level risk: single-stock 8% stoploss, industry 30% cap, portfolio 10% circuit breaker
M8   evaluation/evaluation.py      — Performance metrics (Sharpe, IC decay, attribution)
M9   main.py                       — Daily backtest scheduler (BacktestRunner with checkpoint/resume)
```

**Critical invariant:** All data queries enforce `anchor_date` — no module may access data beyond date T. This prevents forward-looking bias. The `DataManager` is the sole data gateway; nothing calls Qlib's `D` object directly.

### Config

- `quantlab/configs/backtest.yaml` — Main config: rolling window params, pipeline on/off switches (RD-Agent disabled by default), risk thresholds, date range, initial capital.
- `quantlab/configs/kronos_recipes.yaml` — Kronos model recipes and hyperparameters.
- `quantlab/configs/rdagent_evolution.yaml` — RD-Agent evolution loop settings.

## Qlib

### Build & Dev

```bash
cd qlib
pip install -e .[dev]           # Dev mode with all dependencies
make dev                        # Alternative
```

### Test & Lint

```bash
cd qlib/tests
python -m pytest . -m "not slow" --durations=0   # Skip slow tests

cd qlib
make lint       # All linters (black, pylint, flake8, mypy, nbqa)
make black      # Formatting (120 char line limit)
make pylint
make flake8
make mypy
```

### Data Setup

```bash
python scripts/get_data.py qlib_data --target_dir ~/.qlib/qlib_data/cn_data --region cn
```

### Architecture

Layered pipeline: **Data → Model → Strategy → Backtest → Analysis**.

- `qlib/data/` — Core data layer. `D` object is main API (calendar, instruments, features). Expression-based factors with lazy eval. Multi-tier caching (memory → disk). Cython ops in `_libs/`.
- `qlib/contrib/data/` — Alpha158 (158 factors), Alpha360 (360 factors).
- `qlib/model/` — ML training via `Trainer`. Sub-modules: `ens/`, `meta/`, `interpret/`, `riskmodel/`.
- `qlib/strategy/` — Signal-based and rules-based trading strategies.
- `qlib/backtest/` — `Account` (positions/P&L), `Exchange` (market sim), `Executor` (slippage). Nested multi-level execution.
- `qlib/rl/` — RL order execution (Tianshou, PPO/OPDS).
- `qlib/workflow/` — MLflow integration. YAML-based configs (see `examples/benchmarks/`).
- `qlib/cli/` — Run workflows: `qrun <workflow_config.yaml>`

Components are loosely coupled and usable independently.

## Kronos

### Setup & Run

```bash
pip install -r Kronos/requirements.txt
python Kronos/examples/prediction_example.py        # Basic prediction
python Kronos/examples/prediction_batch_example.py   # Batch with uncertainty
```

### Test

```bash
python -m pytest Kronos/tests/ -v
```

### Finetuning

Two paths:

```bash
# Qlib-based (needs CSI300 data):
python finetune/qlib_data_preprocess.py
torchrun --nproc_per_node=2 finetune/train_tokenizer.py
torchrun --nproc_per_node=2 finetune/train_predictor.py
python finetune/qlib_test.py --device cuda:0

# CSV-based (standalone):
python finetune_csv/train_sequential.py --config configs/your_config.yaml
```

### Web UI

```bash
cd Kronos/webui && python run.py   # http://localhost:7070
```

### Architecture

Two-stage pipeline: **Tokenize → Predict**.

- `model/kronos.py` — Three core classes:
  - `KronosTokenizer` — Hierarchical quantization of OHLCV → discrete tokens (Binary Spherical Quantization)
  - `Kronos` — Decoder-only autoregressive Transformer on quantized tokens
  - `KronosPredictor` — High-level API: preprocessing, batch prediction, uncertainty estimation (multi-sample), denormalization
- `model/module.py` — TransformerBlock, BinarySphericalQuantizer, utilities
- `finetune/` — Qlib-integrated finetuning (multi-GPU via torchrun)
- `finetune_csv/` — CSV-based finetuning (no Qlib dependency)

Model zoo (Hugging Face): mini (4.1M), small (24.7M), base (102.3M), large (499.2M).

## RD-Agent

### Build & Dev

```bash
cd RD-Agent
make dev          # All optional deps + pre-commit hook
make install      # Editable install
```

### Test & Lint

```bash
cd RD-Agent
make test-offline           # No API calls needed
make test                   # All tests (fail_under=20%)
make lint                   # mypy, ruff, isort, black, toml-sort
make auto-lint              # Auto-fix
```

Linting targets `rdagent/core` only for mypy/ruff. Black 120 char limit.

### Running

```bash
rdagent health_check
rdagent fin_quant                                  # Factor+model co-evolution
rdagent fin_factor                                 # Factor evolution only
rdagent fin_model                                  # Model evolution only
rdagent fin_factor_report --report-folder=<path>   # Extract factors from reports
rdagent general_model <paper_url>                  # Model from paper
rdagent data_science --competition <name>          # Kaggle
rdagent ui --port 19899 --log-dir <dir>            # Streamlit UI
```

Config via `.env` with LiteLLM backend (OpenAI, Azure, DeepSeek).

### Architecture

Evolutionary loop: **Propose → Code → Run → Evaluate → Evolve**.

- `rdagent/core/` — `Scenario`, `EvoAgent`/`RAGEvoAgent`, `EvolvingStrategy`, `Task`/`Experiment`, `Evaluator`/`Feedback`
- `rdagent/components/` — `coder/`, `proposal/`, `document_reader/`, `knowledge_management/` (RAG), `runner/` (Docker), `workflow/`
- `rdagent/scenarios/` — Domain implementations: `qlib/` (factor/model evolution), `data_science/` (Kaggle), `general_model/` (papers)
- `rdagent/oai/` — LLM integration via LiteLLM
- `rdagent/log/` — Streamlit trace visualization
- `rdagent/app/cli.py` — Typer CLI entry point

---
> Source: [sqyangx/quantlab](https://github.com/sqyangx/quantlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
