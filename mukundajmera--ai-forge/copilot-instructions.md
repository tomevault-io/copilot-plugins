## ai-forge

> Authoritative handbook for automation agents working in the repository. Follow every directive unless superseded by higher-priority instructions. Each top-level directory contains its own `AGENTS.md`; consult the global guide first, then the relevant scoped guide(s) before modifying files.

# AI Forge – Global Agent Handbook

Authoritative handbook for automation agents working in the repository. Follow every directive unless superseded by higher-priority instructions. Each top-level directory contains its own `AGENTS.md`; consult the global guide first, then the relevant scoped guide(s) before modifying files.

---

## Vision & Scope
- Deliver a production-grade, Apple Silicon–optimized LLM fine-tuning stack using PiSSA, QLoRA, RAFT data synthesis, and Ollama deployment.
- Automate the lifecycle end to end: data extraction, training, evaluation, packaging, deployment, monitoring, and autonomous retraining via Repo Guardian.
- Produce verifiable artifacts: curated datasets, fine-tuned checkpoints, GGUF exports, OpenAI-compatible APIs, observability dashboards, and scripted operations.

---

## Directory Map & Handbooks
| Directory | Responsibilities | Handbook |
|-----------|------------------|----------|
| `antigravity_agent/` | Repo Guardian automation, skills, prompts, report templates | [antigravity_agent/AGENTS.md](antigravity_agent/AGENTS.md) |
| `conductor/` | FastAPI service, job queue, Ollama integration, metrics | [conductor/AGENTS.md](conductor/AGENTS.md) |
| `config/` | YAML defaults, preset profiles, configuration helpers | [config/AGENTS.md](config/AGENTS.md) |
| `data_pipeline/` | Tree-sitter mining, RAFT generation, validation | [data_pipeline/AGENTS.md](data_pipeline/AGENTS.md) |
| `docs/` | Canonical documentation set (architecture, configuration, ops) | [docs/AGENTS.md](docs/AGENTS.md) |
| `.github/` | Workflows, issue/PR templates, actions config | [.github/AGENTS.md](.github/AGENTS.md) |
| `judge/` | Evaluation metrics, benchmarks, GGUF exporter, reports | [judge/AGENTS.md](judge/AGENTS.md) |
| `monitoring/` | Prometheus, Grafana, Alertmanager provisioning | [monitoring/AGENTS.md](monitoring/AGENTS.md) |
| `scripts/` | Operational/maintenance scripts referenced by docs | [scripts/AGENTS.md](scripts/AGENTS.md) |
| `tests/` | Unit/integration suites, fixtures, testing README | [tests/AGENTS.md](tests/AGENTS.md) |
| `training/` | Training engine, schemas, callbacks, losses | [training/AGENTS.md](training/AGENTS.md) |
| `research doc/` | Research notes, prompt studies, reference material | [`research doc/AGENTS.md`](research%20doc/AGENTS.md) |

Generated directories (`data/`, `output/`, `logs/`, `models/`, `cache/`, `ai_forge.egg-info/`, `__pycache__/`, `.pytest_cache/`, `.venv/`) lie outside handbook scope; never modify or commit their contents.

---

## Environment & Dependencies
- Python runtime: **3.11+** (constraint in `pyproject.toml`).
- Create virtualenv: `python -m venv .venv && source .venv/bin/activate` (or Windows equivalent).
- Install development extras: `pip install -e ".[dev]"`; use `pip install -e ".[all]"` for UnsLOTH/evaluation features.
- Install pre-commit hooks: `pre-commit install`; run `pre-commit run --all-files` before every commit.
- Configure environment variables: copy `.env.example` → `.env`; set API host/port, directories, Ollama endpoint, optional tokens (`HF_TOKEN`, `WANDB_API_KEY`, `PYPI_API_TOKEN`, `CODECOV_TOKEN`, etc.). Never commit `.env`.
- Apple Silicon validation: `python -c "import torch; print(torch.backends.mps.is_available())"` (must be `True` for GPU acceleration).
- Dependency updates require synchronized changes to `pyproject.toml` and `requirements.txt` plus documentation in `docs/configuration.md`. Adjust Docker assets for new system packages.

---

## Global Standards
- **Formatting**: Black (line length 100). Use Ruff (`I` rule) for import ordering.
- **Linting**: `ruff check .` must be clean. Use focused ignores in `pyproject.toml` only with justification.
- **Typing**: Strict MyPy; provide full type hints. Use `# type: ignore[...]` sparingly with explanatory comment.
- **Docstrings**: Google style enforced by `pydocstyle`. Document parameters, returns, raises for exported APIs.
- **Logging**: Follow patterns in `docs/monitoring.md`. Never log secrets or large payload dumps.
- **Security**: Run Bandit on affected packages when altering security-sensitive areas.
- **Performance**: Profile significant changes to training/data pipeline; summarize findings in PRs.

---

## Mandatory Quality Gates
1. `pre-commit run --all-files`
2. `black .`
3. `ruff check .`
4. `mypy ai_forge data_pipeline training judge conductor antigravity_agent`
5. `bandit -r ai_forge data_pipeline training judge conductor antigravity_agent -ll -ii`
6. `pydocstyle ai_forge`
7. `pytest` (add coverage flags when possible)

Maintain ≥85% coverage (baseline in `tests/README.md`). `pytest.ini` enforces `-x`; override intentionally (`pytest --maxfail=0`) when pursuing full failure reports.

---

## Configuration & Schema Workflow
- Canonical schemas in `training/schemas.py`. For every change:
  1. Update `config/*.yaml` defaults.
  2. Adjust documentation (`docs/configuration.md`, `.env.example` if env-driven).
  3. Add/modify tests covering serialization/deserialization.
- Register new pytest markers in both `pytest.ini` and `[tool.pytest.ini_options].markers` (within `pyproject.toml`).
- Keep tooling configs (`.pre-commit-config.yaml`, `pyproject.toml`) synchronized with documented commands.

---

## Testing Playbook
| Scope | Command | Notes |
|-------|---------|-------|
| Full suite | `pytest` | Uses defaults from `pytest.ini` (`-v`, `--tb=short`, `--strict-markers`, `-x`). |
| Unit only | `pytest tests/unit -m unit -v` | Should complete in <1 minute locally. |
| Integration | `pytest tests/integration -m integration -v` | Requires Ollama at `http://localhost:11434`; mark long cases `slow`. |
| Skip slow | `pytest -m "not slow"` | Mirrors CI behavior. |
| Coverage | `pytest --cov=ai_forge --cov=data_pipeline --cov=training --cov=judge --cov=conductor --cov=antigravity_agent --cov-report=term-missing` | Aim ≥85% coverage and review missing sections. |
| Targeted | e.g., `pytest tests/unit/test_forge.py::TestFineTuneConfig::test_default_config -v` | Use for focused debugging. |

Troubleshooting resources: `docs/troubleshooting.md`, `docs/ci_cd.md`, `tests/README.md`.

---

## Build, Packaging & Deployment
- Build distribution: `python -m build` (wheel + sdist).
- Docker image: `docker build -t ai-forge:latest .` (multi-stage; builder produces wheels, production installs them).
- Docker Compose:
  - Core services: `docker-compose up -d`
  - With monitoring: `docker-compose --profile monitoring up -d`
- Local dev server: `uvicorn conductor.service:app --reload --host 0.0.0.0 --port 8000` (requires active virtualenv and Ollama server).
- Trigger pipeline: `curl -X POST http://localhost:8000/v1/retrain -H "Content-Type: application/json" -d '{"project_path": ".", "force": true}'`.
- Follow `docs/deployment.md` and `docs/production_checklist.md` when shipping changes.

---

## Observability & CI/CD
- Monitoring stack: `docker-compose --profile monitoring up -d` (Prometheus :9090, Grafana :3000, Alertmanager :9093).
- Metrics: `/metrics` endpoint in `conductor/service.py`; maintain `ai_forge_*` namespace. Update Prometheus targets (`monitoring/prometheus.yml`) and alerts (`monitoring/alerts.yml`) together with documentation.
- CI (`.github/workflows/tests.yml`): unit + integration tests, lint/type/docstring checks, Bandit, build artifacts. Runs on pushes/PRs targeting `main` and `develop`.
- Release (`.github/workflows/release.yml`): triggered on `v*` tags, runs macOS full suite, builds artifacts, performs `twine check`, creates release, publishes to PyPI (requires `PYPI_API_TOKEN`).
- Keep README badges aligned with workflow names and supported branches.

---

## Contribution Workflow
- Branching: feature branches from `develop` (preferred) or `main`; rebase before opening PRs.
- Every change must include updated tests, relevant documentation, and commentary on performance/security/ops implications.
- Required reviewer sign-off when touching:
  - `training/` core logic or schemas.
  - `data_pipeline/` extraction/generation heuristics.
  - `conductor/` API contracts or Ollama integration.
  - `antigravity_agent/` automation or skill definitions.
  - `monitoring/` metrics, alerts, dashboards.
- Ensure `docs/production_checklist.md` remains satisfied before labeling work production-ready.

---

## Quick Command Reference
| Task | Command |
|------|---------|
| Bootstrap env | `python -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]"` |
| Install hooks | `pre-commit install` |
| Lint & format | `black . && ruff check .` |
| Type check | `mypy ai_forge data_pipeline training judge conductor antigravity_agent` |
| Security scan | `bandit -r ai_forge data_pipeline training judge conductor antigravity_agent -ll -ii` |
| Run API | `uvicorn conductor.service:app --reload --port 8000` |
| Trigger retrain | `curl -X POST http://localhost:8000/v1/retrain -d '{"project_path": ".", "force": true}'` |
| Unit tests | `pytest tests/unit -m unit -v` |
| Integration tests | `pytest tests/integration -m integration -v` |
| Coverage run | `pytest --cov=ai_forge --cov-report=term-missing` |
| Build package | `python -m build` |
| Docker build | `docker build -t ai-forge:latest .` |
| Compose (core) | `docker-compose up -d` |
| Compose (monitoring) | `docker-compose --profile monitoring up -d` |

---

## Maintaining This Handbook
- Update this document whenever repository structure, tooling, CI/CD processes, or guardrails change.
- Cross-link new directory handbooks as scopes emerge; keep responsibilities unambiguous.
- Validate quick commands/paths after refactors to avoid stale guidance.
- Escalate uncertainties to human maintainers before proceeding with risky operations.

Proceed to the relevant scope handbook(s) before editing module-specific files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mukundajmera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mukundajmera)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
