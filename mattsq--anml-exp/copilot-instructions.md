## anml-exp

> > `anml-exp` is a *pip-installable* anomaly-detection playground with locked

# Agents.md   ― spec _version: **0.3.3**  (2025-07-21)

> **TL;DR**    
> `anml-exp` is a *pip-installable* anomaly-detection playground with locked
> dependencies (`uv.lock`), artefact registry, strict dataset hashing, and a
> structured hardware descriptor in benchmark outputs.  

---

## 0 · Purpose & Non-Goals

This repository remains a **rapid-prototyping and benchmarking framework for
anomaly-scoring / detection algorithms**.

Out of scope:

* Streaming / online detection
* Production serving or long-horizon monitoring
* Adversarial-attack tooling, drift dashboards, model-card generation

---

## 1 · Big Picture

LLM-powered *agents* collaborate with human maintainers to

1. Implement a diverse zoo of anomaly-detection models.  
2. Expose a **uniform API** and artefact registry for seamless benchmarking.  
3. Automate dataset ingestion (SHA-256 verified), metric computation, and
   result logging.  
4. Guard code quality with linting, typing, tests, docs, and a locked
   dependency graph (`uv.lock`).  

---

## 2 · Canonical Folder Layout

.
├── src/
│   └── anml_exp/
│       ├── init.py
│       ├── models/
│       ├── data/
│       ├── benchmarks/
│       ├── registry/            # NEW: model artefact versioning (#43)
│       ├── resources/
│       └── cli.py
├── tests/
├── docs/
├── pyproject.toml
├── uv.lock                       # NEW: reproducible dependency lockfile (#42)
└── README.md

> *Historic note* – the hidden `.agents/` folder mentioned in spec v0.2 has been
> retired; helpers live in `anml_exp/benchmarks/` and `anml_exp/registry/`.

---

## 3 · Common Schema & Interfaces

### 3.1 Base Model API

Every model **must** subclass
`anml_exp.models.base.BaseAnomalyModel` and implement:

| Method / Property | Signature | Notes |
|-------------------|-----------|-------|
| `fit` | `def fit(self, X, y=None) -> Self` | |
| `score_samples` | `def score_samples(self, X) -> NDArray[float]` | Higher ⇒ more anomalous. |
| `predict` | `def predict(self, X, *, threshold=None) -> NDArray[int]` | |
| `decision_threshold` | `@property def decision_threshold(self) -> float` | |
| `save` / `load` | optional | Use `anml_exp.registry` for artefact versioning. |

`anml_exp.registry` stores model binaries and metadata under a semantic version
(`MAJOR.MINOR.PATCH`) with SHA-256 digests.

---

### 3.2 Dataset Registry

```python
from anml_exp.data import load_dataset

X_train, y_train = load_dataset("kddcup99", split="train")

	•	Each dataset module must declare SHA256 hashes for every file.
	•	load_dataset verifies each hash before extraction; mismatch ⇒ HashError
(#44).
	•	Deterministic splits (seed = 42).

⸻

3.3 Metrics & Result Schema

Benchmarks report:
	•	ROC-AUC
	•	PR-AUC (Average Precision)
	•	F1 @ best Youden threshold
	•	Mean wall-time per 1 000 samples

Each run is saved to

results/{exp_name}/{model_name}.json

and must validate against
anml_exp/resources/results-schema.json.

Structured hardware descriptor (#45)

"hardware": {
  "device_type": "GPU",
  "vendor": "NVIDIA",
  "model": "RTX A6000",
  "driver": "535.104",
  "num_devices": 1,
  "notes": "desktop workstation"
}

Minimal example

{
  "$schema": "./results-schema.json",
  "dataset": "kddcup99",
  "model": "isolation_forest",
  "model_version": "0.1.0",
  "n_samples": 145586,
  "seed": 42,
  "hardware": {
    "device_type": "CPU",
    "vendor": "Intel",
    "model": "i7-1185G7",
    "driver": "N/A",
    "num_devices": 1,
    "notes": "laptop"
  },
  "roc_auc": 0.921,
  "pr_auc": 0.604,
  "f1": 0.432,
  "threshold": 0.79,
  "fit_time": 1.23,
  "score_time": 0.02,
  "params": {"n_estimators": 100, "max_samples": "auto"},
  "artefact_digest": "sha256:13f0…"
}


⸻

4 · Agent Roles

Agent	Intent	Success Criteria
Builder	Generate / extend code (models, loaders, registry).	API compliance, passes tests, artefact registered.
Evaluator	Run benchmarks & aggregate metrics.	JSON validates, hardware descriptor correct.
Reviewer	Static analysis, typing, docs, tests, perf.	CI green (ruff, mypy, pytest, hash check, lock diff).


⸻

5 · Contribution Workflow

flowchart TD
    draft["Builder → Draft PR"]
    review["Reviewer → CI checks"]
    maintainer["Human → Merge / Request changes"]
    draft --> review --> maintainer

CI additionally ensures:
	•	uv sync --frozen produces identical env (#42).
	•	Dataset SHA-256s match declared values (#44).

⸻

6 · Coding Standards
	•	Dependency lock: uv.lock is the single source of truth.
	•	PEP 8 via ruff; PEP 561 typing (mypy --strict).
        •       Speed up mypy in CI by caching `.mypy_cache` and installing
                `mypy[faster-cache]` via `uv pip` to ensure the local
                environment runs the optimized wheels.
	•	NumPy-style docstrings.
	•	pyproject.toml + uv.lock define mandatory and optional extras.

⸻

7 · Testing Strategy
	•	Unit + property tests.
	•	Hash-verification tests for every dataset file.
	•	CI fails if uv lock --check detects drift.
	•	Perf suite (tests/perf/) skipped in CI.

⸻

8 · Installation & Quick-Start

# Reproducible dev install
uv sync --frozen
pip install -e ".[torch,plot]"
# After release:
pip install anml-exp[torch,plot]

CLI:

anml-exp benchmark --dataset toy-blobs \
                   --model isolation_forest \
                   --output results/demo.json


⸻

9 · Road-Map

Milestone	Owner	Exit Criteria
M0 – Skeleton	Builder	Base class, dataset registry (SHA-256), artefact registry, CI, uv.lock.
M1 – Classical Benchmark	Evaluator	3 tabular datasets; JSON outputs pass new schema.
M2 – Deep Models	Builder	AutoEncoder, DeepSVDD, USAD registered & versioned.
M3 – Time-Series Support	Builder + Evaluator	Loader + STOMP baseline + benchmarks.


⸻

10 · Open Questions
	1.	Unified config system (omegaconf) – still pending.
	2.	Preferred experiment tracker (mlflow, wandb, plain JSON).
	3.	CPU vs GPU determinism in CI.
	4.	Sandboxing policy for code-gen agents.

⸻

11 · Meta
	•	_spec_version bumped → 0.3.3 (adds #42–#45).
	•	See CONTRIBUTING.md for human-targeted guidelines.
	•	results-schema.json is the machine-readable contract.

Last updated – 2025-07-20 @ 20:55 AEST

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattsq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mattsq)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
