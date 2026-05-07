## prowl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prowl is an autonomous vulnerability discovery and exploit validation CLI tool. It decomposes security research into structured stages: reconnaissance (deterministic, tree-sitter-based), hypothesis generation (LLM Layer 1), triage and chain analysis (LLM Layer 2), and proof-of-concept validation in sandboxed Docker containers (LLM Layer 3).

The full specification is in `prowl.md`. The phased implementation plan is in `plan.md`.

## Tech Stack

- **Python 3.11+** with `pyproject.toml` (PEP 621)
- **Key dependencies:** `langchain-core` + provider packages (multi-provider LLM), `tree-sitter` + `tree-sitter-language-pack`, `docker` (docker-py), `pydantic` v2, `click`, `rich`, `anyio`, `PyYAML`
- **LLM providers:** OpenAI (`langchain-openai`), Anthropic (`langchain-anthropic`), Google (`langchain-google-genai`), Ollama (`langchain-ollama`)
- **Dev tools:** `pytest` + `pytest-asyncio`, `ruff`, `mypy`, `coverage`
- **Package layout:** `src/prowl/` (src layout)

## Build & Development Commands

```bash
# Install (editable, with dev dependencies + Anthropic provider)
pip install -e ".[dev,anthropic]"

# Or install all LLM providers
pip install -e ".[dev,all-llm]"

# Or set PYTHONPATH manually
export PYTHONPATH=src

# Run the CLI
python -m prowl
python -m prowl scan <path>
python -m prowl scan --format json <path>
python -m prowl scan --format markdown -o report.md <path>
python -m prowl scan --categories memory,auth <path>

# Linting
ruff check src/prowl/
ruff check src/prowl/ --fix   # auto-fix import sorting etc.

# Run all tests (385 tests, ~1s)
pytest tests/ -v --ignore=tests/fixtures

# Run tests for a specific module
pytest tests/test_recon/ -v
pytest tests/test_context_builder/ -v

# Run a single test
pytest tests/test_recon/test_signals.py::TestSqlInjectionSignals -v

# Run only Docker-dependent tests
pytest tests/ -v -m docker

# Skip Docker-dependent tests
pytest tests/ -v -m "not docker"

# Run with coverage
coverage run -m pytest tests/ && coverage report
```

## Architecture

### Pipeline Stages

```
Codebase → Recon (deterministic) → Layer 1: Hypothesis (LLM) → Layer 2: Triage + Chains (LLM) → Layer 3: PoC Validation (LLM + sandbox) → Report
```

- **Recon** (`src/prowl/recon/`): Tree-sitter parsing, function extraction, risk signal detection, vulnerability scoring, call graph construction, taint tracking, target prioritization. Fully deterministic, no LLM.
- **Context Builder** (`src/prowl/context_builder/`): Assembles structured context chunks per target function for each layer (~4K tokens L1/L2, ~8K tokens L3). Deterministic.
- **Hypothesis Engine** (`src/prowl/hypothesis/`): Layer 1 — parallel LLM calls over scored targets, confidence gating (>=0.7 promote, 0.4-0.7 batch, <0.4 suppress).
- **Triage** (`src/prowl/triage/`): Layer 2 — classifies hypotheses as exploitable/mitigated/FP/uncertain. Deterministic chain grouping followed by LLM chain evaluation.
- **Validation** (`src/prowl/validation/`): Layer 3 — [Claw Code](https://github.com/ultraworkers/claw-code) agentic validation inside Docker sandbox. `ClawValidationBackend` in `claw_backend.py` builds the actual target project (not standalone PoC code): C/C++ compiled with ASAN via cmake/autotools/meson/make, Python/Node servers started and hit with crafted requests, Go/Rust/Java built and run with crafted inputs. Per-vuln-class result checking. Patch generation.
- **Sandbox** (`src/prowl/sandbox/`): Docker container lifecycle with hardened security policy. Bridge network (for Claw LLM API access), writable filesystem, capability-dropped, no-new-privileges, resource limits. Enriched per-language images with full build toolchains. ASAN/UBSAN instrumentation for C/C++.
- **Pipeline Orchestrator** (`src/prowl/pipeline/`): Wires stages together, manages budget gating, scan resumability, and error handling (per-target graceful degradation).

### Key Design Patterns

- **All LLM calls go through LangChain** (`src/prowl/llm/langchain_client.py`). The `LLMClient` protocol (`src/prowl/llm/sampling.py`) defines typed methods per layer. `LangChainClient` implements it with support for OpenAI, Anthropic, Google, and Ollama providers. Per-layer model routing is supported via config.
- **Structured output enforcement**: Pydantic `model_json_schema()` in prompts, `model_validate_json()` for responses. `ValidationError` triggers one retry with error context; second failure skips the target.
- **Dependency injection**: `LLMClient` and `SandboxManager` are injected via constructors. Tests swap in `MockLLMClient` and `MockSandboxManager`.
- **Async architecture**: `anyio` for bounded parallelism (`CapacityLimiter`). Docker-py ops wrapped in `asyncio.to_thread()`. Concurrency limits: 8 hypothesis, 4 triage, 2 validation.
- **All data models are Pydantic v2** (`src/prowl/models/`).

### State & Caching

- Runtime state in `.prowl/` (gitignored except `suppressions.json` and `missed.json`)
- Cache keys: `hash(function_content + caller_interface_signature + rubric_version)`
- Cross-cutting invalidation on middleware/framework/dependency changes
- Scan state persisted in `.prowl/scan-state/` for resumability
- Custom rubrics in `.prowl/rubrics/` extend (don't replace) built-in rubrics

## Testing

385 tests across 19 test files, running in about 1 second. Test structure:

- `tests/test_recon/` — Parser, exclusions, extractor, signals, scorer, call graph (pure unit tests against fixtures)
- `tests/test_context_builder/` — Context assembly for all 3 layers
- `tests/test_hypothesis/` — Engine with MockLLMClient, confidence gating
- `tests/test_triage/` — Classifier, chain analyzer grouping logic
- `tests/test_cache/` — Store put/get/invalidate/persistence
- `tests/test_suppression/` — CRUD, scope matching, orphan detection
- `tests/test_output/` — All 5 output formats (text, JSON, SARIF, AI, Markdown)
- `tests/test_pipeline/` — Full orchestrator integration with mocked LLM + sandbox
- `tests/test_llm/` — LangChain client, provider routing, config loading
- `tests/test_integration/` — End-to-end recon on fixture codebases

Test fixtures in `tests/fixtures/`:
- `python_app/` — Flask app with SQLi, IDOR, missing auth, XSS, command injection
- `c_project/` — C code with buffer overflow, use-after-free, integer overflow, format string
- `node_app/` — Express app with command injection, XSS, SQLi

Mocks in `tests/conftest.py`: `MockLLMClient` and `MockSandboxManager` allow testing all pipeline phases without real LLM/Docker.

## Configuration

Project-level config is `prowl.yml` in the project root. Key sections: `scan`, `reconnaissance`, `scoring`, `triage`, `validation`, `sandbox`, `concurrency`, `budget`, `cache`, `output`, `resume`, `llm`. See spec for full schema.

---
> Source: [toniantunovi/prowl](https://github.com/toniantunovi/prowl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
