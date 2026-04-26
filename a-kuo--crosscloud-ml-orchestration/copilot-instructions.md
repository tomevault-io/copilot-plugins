## crosscloud-ml-orchestration

> Multi-cloud ML inference router that uses **Shannon entropy of transformer attention distributions** to route requests between GCP Cloud Run (fast/confident path) and AWS SageMaker (heavy/uncertain path). Built as a portfolio-grade demonstration of information-theoretic ML serving.

# CLAUDE.md — Cross-Cloud ML Inference Orchestration

## Project Identity

Multi-cloud ML inference router that uses **Shannon entropy of transformer attention distributions** to route requests between GCP Cloud Run (fast/confident path) and AWS SageMaker (heavy/uncertain path). Built as a portfolio-grade demonstration of information-theoretic ML serving.

**Author:** Austin Kuo (aus.kuo03@gmail.com)

## Architecture at a Glance

```
Input text
  └─► AttentionEntropyProbe (DistilBERT)
        └─► H_route = mean Shannon entropy across final N layers × all heads
              ├── H < τ  →  GCP Cloud Run  (BERT ABSA, fast path)
              └── H ≥ τ  →  AWS SageMaker  (hallucination scorer, heavy path)

Nightly Airflow DAG:
  entropy_audit → [threshold_recalibrate, retrain_trigger] → health_check
```

## Repository Layout

| Path | Purpose |
|------|---------|
| `router/entropy.py` | `AttentionEntropyProbe` — extracts attention tensors, computes per-head Shannon entropy, aggregates to H_route |
| `router/threshold.py` | `ThresholdCalibrator` — isotonic regression on (entropy, error) pairs to produce tau; save/load JSON; AUROC reporting |
| `router/router.py` | `InferenceRouter` — combines probe + calibrator, emits `RoutingDecision` dataclass (BigQuery-serialisable) |
| `inference/server.py` | FastAPI server with `/infer`, `/entropy`, `/health`; controlled by `SERVE_TARGET` env var |
| `inference/models.py` | HuggingFace pipeline wrappers for ABSA and hallucination detection |
| `inference/Dockerfile` | Multi-target build: `router`, `absa`, or `hallucination` mode |
| `airflow/dags/crosscloud_orchestration.py` | 4-task DAG: entropy audit (KL divergence), threshold recalibration, retrain trigger, health check |
| `telemetry/bigquery_schema.json` | `routing_events` table schema |
| `telemetry/sample_telemetry.py` | Synthetic JSONL generator |
| `tests/` | pytest suite — 34 fast tests + 4 slow (model-loading) tests behind `--runslow` |

## Key Mathematical Concepts

These are central to the codebase and must be preserved in any modifications:

- **Attention entropy:** `H(α) = -Σ p(a_i) log p(a_i)` computed over softmax-normalised attention weights. High H = uniform attention = model uncertainty. Low H = focused attention = confidence.
- **H_route aggregation:** Mean entropy across all heads in the final N transformer layers. This is the scalar routing signal.
- **Threshold tau (τ):** Not a fixed constant — calibrated nightly via isotonic regression on `(H_route, is_error)` pairs. Tau is the minimum H where `P(error|H) ≥ ε` (risk tolerance, default 0.15).
- **KL divergence drift detection:** `D_KL(P_current || Q_baseline)` on entropy histograms. >0.1 nats triggers recalibration; >0.3 nats triggers full retraining.
- **Numerical stability:** All entropy computations clip probabilities to `[1e-12, 1.0]` before log.

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run fast tests (no model download)
pytest

# Run all tests including DistilBERT model tests
pytest --runslow

# Run the inference server locally (router mode, downloads DistilBERT on first run)
cd inference && uvicorn server:app --host 0.0.0.0 --port 8080 --reload

# Generate sample telemetry
python telemetry/sample_telemetry.py --rows 2000 --out telemetry/sample_rows.jsonl

# Run Airflow DAG locally (mocked cloud)
export AIRFLOW_HOME=$(pwd)/airflow MOCK_CLOUD=true
airflow db init && airflow dags test crosscloud_ml_orchestration 2026-03-18

# Build Docker image
docker build -f inference/Dockerfile -t crosscloud-router .
```

## Coding Conventions

- **Python 3.11+**, type hints everywhere (`from __future__ import annotations`).
- **Dataclasses** for data containers (`EntropyResult`, `RoutingDecision`, `CalibrationResult`) — all expose `.to_dict()` for JSON serialisation.
- **Structured logging** via `structlog` in the server, stdlib `logging` in library code.
- **No hardcoded thresholds** — tau, risk tolerance, KL bounds, SLA limits are all configurable via constructor args or env vars.
- **Mock-friendly design** — `MOCK_CLOUD=true` env var gates all real cloud SDK calls in the Airflow DAG. Router tests use mock probes (no model download).
- **Tests:** Fast unit tests test pure functions and mocked probes. Slow integration tests (behind `@pytest.mark.slow` / `--runslow`) load real DistilBERT.
- Docstrings on all public classes and functions. Module-level docstrings explain purpose and mathematical grounding.
- No comments that simply narrate code. Comments only for non-obvious mathematical relationships or stability concerns.

## Environment Variables

| Variable | Used In | Default | Purpose |
|----------|---------|---------|---------|
| `SERVE_TARGET` | server.py | `router` | `router`, `absa`, or `hallucination` |
| `TAU` | server.py | (calibrator default 2.0) | Override routing threshold |
| `PROBE_MODEL` | server.py | `distilbert-base-uncased` | HuggingFace model ID |
| `MOCK_CLOUD` | Airflow DAG | `true` | Gate real GCP/AWS SDK calls |
| `BQ_PROJECT` | Airflow DAG | `crosscloud-demo` | BigQuery project ID |
| `KL_RECALIBRATE_BOUND` | Airflow DAG | `0.1` | Nats threshold for recalibration |
| `KL_RETRAIN_BOUND` | Airflow DAG | `0.3` | Nats threshold for full retrain |
| `LATENCY_SLA_MS` | Airflow DAG | `150.0` | p99 latency SLA for health check |

## Scope Boundaries

This is a **portfolio project**. In scope: entropy module + tests + AUROC benchmarks, Dockerised server, Airflow DAG with mocked hooks, BigQuery schema + synthetic data. Out of scope: real IAM, production data, load testing. Do not add features that claim production readiness that does not exist — document limitations honestly instead.

## Common Agent Tasks

- **Adding a new routing destination:** Add enum value to `RoutingDestination` in `router/router.py`, add model wrapper in `inference/models.py`, update routing logic in `router/router.py`, add server handler in `inference/server.py`.
- **Changing the probe model:** Update default in `AttentionEntropyProbe.__init__()`, update `PROBE_MODEL` references in Dockerfile ARG and README. Run `pytest --runslow` to validate.
- **Adjusting drift bounds:** Change `KL_RECALIBRATE_BOUND` / `KL_RETRAIN_BOUND` in the DAG or via env vars. Document the empirical rationale.
- **Adding a new Airflow task:** Define callable inside the `with DAG` block, create `PythonOperator`, wire into the dependency graph after `t_audit`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/A-Kuo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
