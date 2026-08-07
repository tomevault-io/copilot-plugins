## contextops

> > Instructions for autonomous coding agents contributing to ContextOps.

# AGENTS.md

> Instructions for autonomous coding agents contributing to ContextOps.

ContextOps is a deterministic structural analysis tool for LLM context. Every contribution must preserve that identity.

This document describes the project's architectural constraints, engineering philosophy, and contribution expectations for both human contributors and autonomous coding agents.

---

## Mission

ContextOps provides **deterministic static analysis** for LLM context **before inference**.

Its purpose is to measure the structural health of context — not the quality of prompts, reasoning, retrieval, or model outputs.

Every change should strengthen this mission.

---

## Core Design Principles

### 1. Determinism Is Non-Negotiable

Given identical input, ContextOps must always produce:

- Identical score
- Identical findings
- Identical recommendations
- Identical JSON output

**Never introduce:**

- Randomness or random seeds in the analysis path
- Sampling or probabilistic scoring
- Time-dependent behavior (`datetime.now()` in scoring logic)
- Network-dependent behavior
- GPU-dependent behavior

The one intentional exception is `contextops/core/roast.py` — roasts are explicitly non-deterministic by design and are excluded from the stability contract. Do not apply determinism constraints to roast selection.

---

### 2. Model Independence

ContextOps must never require:

- LLM inference of any kind
- Embedding models (dense or sparse)
- Vector databases
- GPUs or accelerators
- External API calls in the analysis pipeline

The core analysis engine (`contextops/core/`, `contextops/analyzers/`) must remain **pure Python**.

Optional integrations may exist in `contextops/integrations/` and must not be imported by the core engine.

---

### 3. Structural Scope Only

ContextOps evaluates **only structural properties** of context:

- **Redundancy** — duplicate or near-duplicate chunks
- **Density** — format overhead, whitespace waste, entropy compression
- **Structure** — context type imbalance (system, retrieval, memory, tools)
- **Concentration** — over-reliance on a small number of sources

It intentionally does **not** evaluate:

- Semantic similarity or meaning
- Prompt quality or intent
- Hallucinations or factual correctness
- Reasoning quality
- Retrieval relevance
- Output correctness

**Do not expand the scoring engine beyond structural analysis.** If a proposed feature requires understanding what the context means, it is out of scope.

---

### 4. Scoring Stability Is a Contract

Scoring changes are **breaking changes**. Before modifying:

- Scoring formulas (`contextops/core/engine.py`)
- Penalty weights or thresholds
- Aggregation or normalization logic
- Default config values in `contextops/core/config.py`

...check `STABILITY.md` to understand version implications. Changes that alter output for identical input require at minimum a **minor version bump**; formula changes require a **major version bump**.

Run the signal contract tests (`tests/test_signal_contract.py`) and stability tests before and after any engine change.

---

## Architecture

```
Input (JSON / dict / list / str)
        │
        ▼
  Normalizer  (contextops/core/normalizer.py)
        │
        ▼
  ContextBundle  (contextops/core/models.py)
        │
        ▼
  Analyzers  (contextops/analyzers/)
   ├── tokens.py        — token counting per type
   ├── redundancy.py    — LSH + Jaccard deduplication
   ├── density.py       — format overhead / entropy compression (shadow metric)
   └── structure.py     — context type ratio analysis
        │
        ▼
  Score Engine  (contextops/core/engine.py)
   ├── _calc_redundancy_penalty()   — 0–30 pts
   ├── _calc_density_penalty()      — 0–30 pts
   ├── _calc_structure_penalty()    — 0–20 pts
   └── _calc_concentration_penalty()— 0–20 pts
        │
        ▼
  Recommendations  (engine._generate_recommendations())
        │
        ▼
  AnalysisResult  (contextops/core/models.py)
        │
        ├── CLI Renderer  (contextops/cli/renderer.py)
        └── JSON Output   (AnalysisResult.to_dict())
```

**Keep analyzers independent.** An analyzer must read only its designated inputs (see the signal contract in `STABILITY.md`). Cross-axis reading is prohibited:

- `redundancy.py` reads only the `ContextBundle`
- `density.py` reads only raw text / structural properties
- `structure.py` reads only token ratios by type
- `concentration_penalty` in the engine reads only source metadata

---

## File Map

| Path | Purpose |
|------|---------|
| `contextops/core/engine.py` | Central orchestrator and scoring |
| `contextops/core/models.py` | All data models (`ContextBundle`, `AnalysisResult`, etc.) |
| `contextops/core/normalizer.py` | Input parsing into `ContextBundle` |
| `contextops/core/config.py` | Config schema and archetype presets |
| `contextops/core/roast.py` | Non-deterministic roast engine (excluded from stability) |
| `contextops/core/telemetry.py` | Local-only event logging |
| `contextops/analyzers/redundancy.py` | LSH-based redundancy detection |
| `contextops/analyzers/density.py` | Structural density signal |
| `contextops/analyzers/structure.py` | Context type ratio analysis |
| `contextops/analyzers/tokens.py` | Token counting via tiktoken |
| `contextops/api/inspect.py` | Public Python API (`inspect_context()`) |
| `contextops/api/stability.py` | Determinism verification API |
| `contextops/api/diff.py` | Context diff API |
| `contextops/cli/main.py` | CLI commands (inspect, check, demo, diff, stability, badge) |
| `contextops/cli/renderer.py` | Terminal rendering — view layer only |
| `contextops/integrations/` | Optional integrations, never imported by core |
| `tests/` | Full test suite (211 tests) |

---

## Performance

Performance matters. Do not regress it silently.

**Never introduce:**
- O(N³) pairwise algorithms without a gating mechanism
- Repeated tokenization of the same content
- Repeated parsing of the same payload
- Unnecessary allocations in hot paths

**Prefer:**
- Linear passes over the bundle
- Cached computations (LSH buckets, token counts)
- Immutable data structures where practical

**Current performance targets:**

| Payload size | Target latency |
|---|---|
| 5k tokens | < 2 seconds |
| 20k tokens | < 5 seconds |
| 50k tokens | < 10 seconds |

If a change is expected to significantly alter performance, include a benchmark run using `tests/test_benchmarks.py`.

---

## Dependencies

The analysis engine must stay lightweight.

**Allowed core dependencies (declared in `pyproject.toml`):**
- `tiktoken` — token counting
- `click` — CLI

**Do not add** heavyweight ML frameworks, embedding libraries, or vector stores to core dependencies.

Optional integrations (e.g. LangChain, LlamaIndex) belong in `contextops/integrations/` with corresponding optional dependency declarations.

---

## CLI Philosophy

The CLI is a **view layer**. JSON is the source of truth.

- `contextops inspect` — analyze and pretty-print
- `contextops check` — CI gate (exit code 0/1)
- `contextops demo` — built-in showcase
- `contextops diff` — compare two contexts
- `contextops stability` — determinism verification
- `contextops badge` — shields.io badge generation
- `contextops telemetry` — local event log management

**Rules for CLI changes:**
- Do not remove or rename existing flags
- New flags must have safe defaults that preserve existing behavior
- Machine-readable output belongs in `--json-output` mode
- Human-readable output belongs in the terminal renderer
- The renderer (`contextops/cli/renderer.py`) must never contain scoring logic

---

## Public API Contract

The Python API (`contextops/api/inspect.py`) is considered stable.

```python
from contextops.api.inspect import inspect_context

result = inspect_context(payload)
result.score          # int 0–100
result.to_dict()      # stable JSON schema
```

**Never break:**
- `inspect_context()` signature (add optional parameters only)
- `AnalysisResult.score` type or semantics
- `AnalysisResult.to_dict()` output schema
- `schema_version` field in JSON output (currently `"2.1"`)

When adding new fields to `AnalysisResult.to_dict()`, they must be **additive** and **optional** to parse.

---

## Error Handling

Prefer **explicit errors** over silent recovery.

Every public API should fail predictably and clearly. Error messages must explain:
1. **What** failed
2. **Why** it failed
3. **How to fix it**

Do not swallow exceptions in the analysis path. Silent failure in a linting tool is worse than a crash.

---

## Testing Expectations

All contributions must include or update tests.

**Test files and what they cover:**

| File | Covers |
|------|--------|
| `test_api.py` | Public API contracts, edge cases |
| `test_signal_contract.py` | Cross-axis isolation, determinism |
| `test_normalizer.py` | All supported input formats |
| `test_redundancy.py` | LSH accuracy, Jaccard thresholds |
| `test_density.py` | Density signal computation |
| `test_structure.py` | Structural penalty logic |
| `test_tokens.py` | Token counting accuracy |
| `test_chaos.py` | Adversarial and malformed inputs |
| `test_archetypes.py` | Archetype resolution priority chain |
| `test_benchmarks.py` | Performance regression |
| `test_cli_commands.py` | CLI command smoke tests |
| `test_telemetry.py` | Local event logging |

**Run the full suite before submitting:**
```bash
python -m pytest
```

All 211 tests must pass. If a scoring algorithm changes, add or update regression tests in `test_signal_contract.py`.

---

## Documentation

Documentation should describe:
- **What** the feature does
- **Why** it exists
- **Limitations** and known edge cases

Avoid marketing language. Prefer precise engineering language.

Every new feature should document:
- Purpose and scope
- Constraints (what it does NOT do)
- Expected behavior
- Known limitations

Relevant docs in the repository:

| File | Purpose |
|------|---------|
| `README.md` | User-facing overview and quickstart |
| `STABILITY.md` | Versioning and behavioral guarantees |
| `CONTRIBUTING.md` | Human contributor guide |
| `HIGH_LEVEL_DOC.md` | Methodology and algorithm details |
| `USER_GUIDE.md` | End-user CLI reference |
| `CHANGELOG.md` | Version history |

---

## What ContextOps Is Not

Do not evolve ContextOps into:

- An LLM evaluator
- A prompt optimizer
- A semantic search engine
- An embedding framework
- A hallucination detector
- An AI safety framework
- A retrieval quality judge

Those are valuable problems. They are out of scope for this project.

---

## Preferred Contributions

Good contributions include:

- Improved structural analyzers (new penalties, better accuracy)
- Faster algorithms that preserve determinism
- Reduced false positives in redundancy detection
- Clearer diagnostics and recommendations
- Better CLI UX (without breaking existing flags)
- New integrations in `contextops/integrations/`
- Benchmark improvements and regression tests
- Documentation improvements
- More brutal roasts in `contextops/core/roast.py`

---

## Philosophy

Modern software engineering relies on deterministic tooling:

- Compilers catch syntax errors.
- Linters catch code smells.
- Formatters enforce consistency.
- Static analyzers detect structural problems.

LLM applications deserve the same engineering discipline.

ContextOps exists to make context quality **observable**, **measurable**, and **testable** before inference.

Every contribution should reinforce that goal.

---
> Source: [Abhijeet777ui/contextops](https://github.com/Abhijeet777ui/contextops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
