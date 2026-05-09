## pydantic-collab

> Core runtime code lives in `pydantic_collab/`. `collab.py` holds the orchestration engine, `_types.py` defines settings and runtime state, `_utils.py` contains prompt/context helpers, and `custom_collabs.py` provides ready-made topologies (forward chains, stars, meshes). Public docs live in `README.md` and `docs/topology_example.png`. `examples/` contains numbered Python scripts that demonstrate progressively more complex agent graphs and reuse the helper utilities in `examples/example_tools.py`. Automated checks sit under `tests/` and mirror real collaboration scenarios (handoff control, topology validation, prompt output, import safety). Generated builds go to `dist/`, and `.logfire/` plus `.ruff_cache/` store tooling artifacts.

# Repository Guidelines

## Project Structure & Module Organization
Core runtime code lives in `pydantic_collab/`. `collab.py` holds the orchestration engine, `_types.py` defines settings and runtime state, `_utils.py` contains prompt/context helpers, and `custom_collabs.py` provides ready-made topologies (forward chains, stars, meshes). Public docs live in `README.md` and `docs/topology_example.png`. `examples/` contains numbered Python scripts that demonstrate progressively more complex agent graphs and reuse the helper utilities in `examples/example_tools.py`. Automated checks sit under `tests/` and mirror real collaboration scenarios (handoff control, topology validation, prompt output, import safety). Generated builds go to `dist/`, and `.logfire/` plus `.ruff_cache/` store tooling artifacts.

## Build, Test, and Development Commands

```bash
# Install & lock dependencies using uv.lock
uv sync

# Run the full pytest suite (includes asyncio-based scenarios)
uv run pytest

# Execute an example topology with the configured environment variables
uv run --env-file .env examples/01_simple_chain.py
```

## Coding Style & Naming Conventions
- **Indentation**: 4 spaces everywhere; tabs are not used.
- **File naming**: Modules follow `snake_case` (e.g., `_utils.py`, `custom_collabs.py`); tests use `test_*.py`.
- **Function/variable naming**: Snake case for functions (`default_build_agent_prompt`), PascalCase only for classes like `CollabAgent`.
- **Linting**: Ruff enforces a 120-character limit, Google-style docstrings, single quotes, import sorting, and extra rules (`Q`, `UP`, `I`, `D`, etc.). Run `uv run ruff check .` and `uv run ruff format .` before opening a PR. Pyright strict mode guards types: `uv run pyright`.

## Testing Guidelines
- **Framework**: Pytest with `pytest-asyncio` for async agents; tests occasionally use `pydantic_ai.models.test.TestModel` to stub LLMs.
- **Test files**: Located in `tests/`, following `test_<feature>.py` and using shared fixtures from `tests/fixtures_handoff.py`.
- **Running tests**: `uv run pytest` (ensure `.env` is present when tests depend on API keys).
- **Coverage**: No explicit threshold in repo—focus on exercising handoff paths, topology validation, and prompt builders before submissions.

## Commit & Pull Request Guidelines
- **Commit format**: History is minimal (`First Commit`), so follow conventional, imperative summaries such as `Add topology validation tests` or `Fix Collab handoff recursion`. Keep scope tight; run lint/type/test checks before committing.
- **PR process**: Include what topology or subsystem you touched, proof of local test runs, and screenshots/text for visualization changes. Reference relevant example scripts when adding new behaviors.
- **Branch naming**: Not predefined—use descriptive branches like `feature/handoff-context` or `fix/prompt-builder` so reviewers can infer scope quickly.

---

# Repository Tour

## 🎯 What This Repository Does

`pydantic-collab` is a declarative multi-agent orchestration framework for [pydantic-ai](https://ai.pydantic.dev/). It lets you design agent topologies (chains, stars, meshes, or fully custom graphs), govern tool-call vs. handoff behavior, and observe execution history with rich context.

**Key responsibilities:**
- Normalize arbitrary agent declarations into a validated collaboration graph
- Execute agents sequentially or via tool calls while enforcing limits (handoffs, call depth, usage)
- Provide ergonomic APIs for prompt/context customization, visualization, and testing

---

## 🏗️ Architecture Overview

### System Context
```
Client code (examples/tests) → Collab orchestration layer → pydantic-ai agents
                                       ↓
                                Optional observers (logfire, visualizers)
```

### Key Components
- **`Collab` (pydantic_collab/collab.py)** – Central orchestrator; validates graphs, manages handoffs/tool calls, tracks execution history, and exposes sync/async run APIs.
- **`CollabAgent` & settings (pydantic_collab/_types.py)** – Strongly-typed wrappers describing each agent’s capabilities, limits, and state; includes `CollabSettings`, `HandOffBase`, `CollabRunResult`, and runtime `CollabState`.
- **Prompt/context utilities (pydantic_collab/_utils.py)** – Convert message histories to text, extract agent-to-agent tool calls, and build descriptive prompts via `default_build_agent_prompt`.
- **Specialized topologies (pydantic_collab/custom_collabs.py)** – `PipelineCollab`, `StarCollab`, `MeshCollab`, and `HierarchyCollab` preconfigure `_build_topology` to reduce boilerplate.

### Data Flow
1. User declares `Collab` (or a custom subclass) with agent tuples or `CollabAgent` objects.
2. `_build_topology` normalizes names, allowed tool targets, and handoff routes; validation ensures final agents are reachable and no dead ends remain.
3. `run`/`run_sync` executes the entry agent. Agents may:
   - Produce final output (captured in `CollabRunResult`).
   - Call peers as tools (synchronous, depth-limited by `max_agent_call_depth`).
   - Emit `HandOffBase` payloads to transfer control, with optional reasoning/context flags.
4. `CollabState` records execution history, reasoning, agent contexts, and usage. Optional logfire instrumentation and visualization hooks observe the run.

---

## 📁 Project Structure [Partial Directory Tree]

```
agent_collab_poc/
├── AGENTS.md                # Contributor & architecture doc (this file)
├── README.md                # Quick-start, examples, visualization guidance
├── pyproject.toml           # Project metadata, dependencies, Ruff & Pyright settings
├── uv.lock                  # Frozen dependency graph for uv
├── docs/
│   └── topology_example.png # Reference image for graph visualization
├── examples/
│   ├── 01_simple_chain.py   # Forward handoff basics
│   ├── ...
│   └── example_tools.py     # Shared tools for demos
├── pydantic_collab/
│   ├── __init__.py          # Public package exports
│   ├── collab.py            # Core orchestration logic
│   ├── custom_collabs.py    # Star, mesh, forward, hierarchy helpers
│   ├── _types.py            # Dataclasses & settings for agents/runs
│   └── _utils.py            # Prompt & context helpers
├── tests/
│   ├── fixtures_handoff.py  # Shared test fixtures
│   ├── test_topology_validation.py
│   └── ...
```

### Key Files to Know

| File | Purpose | When You'd Touch It |
|------|---------|---------------------|
| `pydantic_collab/collab.py` | Implements `Collab` with topology normalization, run loop, and history tracking | Extending orchestration behavior, enforcing new limits, or instrumenting runs |
| `pydantic_collab/_types.py` | Houses `CollabAgent`, `CollabSettings`, `CollabState`, `HandOffBase`, and result structures | Changing how agents are described, how context is stored, or adjusting settings validation |
| `pydantic_collab/_utils.py` | Builds prompts and handoff context, extracts tool-call transcripts | Updating prompt guidance, altering context-sharing rules, or adding new diagnostics |
| `pydantic_collab/custom_collabs.py` | Ready-made subclasses (`PipelineCollab`, `StarCollab`, `MeshCollab`, `HierarchyCollab`) | Adding a new topology preset or tweaking `_build_topology` logic |
| `examples/0*.py` | Progressive demos covering chains, meshes, parallel fan-out, tool routing, visualization, etc. | Reproduce bugs, document new capabilities, or provide sample topologies |
| `examples/example_tools.py` | Shared tool implementations used across demos | Add reusable tools or dependencies surfaced in examples |
| `tests/test_topology_validation.py` | Ensures every agent can reach the final output and detects cycles/dead ends | When editing validation rules or `_build_topology` internals |
| `tests/test_handoff_data_control.py` | Verifies how reasoning, transformed queries, and multi-step pipelines propagate via `HandOffBase` | Modifying handoff payloads or context inclusion controls |
| `tests/test_prompt_output.py` | Smoke test for `default_build_agent_prompt` under different capability mixes | Changing prompt formatting or instructions |
| `tests/test_imports.py` | Guards package import surface | When adding exports to `__init__.py` |
| `pyproject.toml` | Defines dependencies, optional extras (`viz`, `logfire`, `examples`), and Ruff/Pyright settings | Adding dependencies, updating linters, or adjusting build metadata |

---

## 🔧 Technology Stack

### Core Technologies
- **Language:** Python ≥ 3.11 (per `requires-python`) – needed for dataclass features, `asyncio`, and modern typing used throughout the orchestrator.
- **Framework:** [pydantic-ai](https://ai.pydantic.dev/) – supplies `Agent`, model abstractions, toolsets, and `TestModel` stubs for deterministic testing.
- **Orchestration Layer:** Custom `Collab` engine built atop Pydantic dataclasses & `asyncio` primitives.
- **Optional Visualization:** `graphviz`, `matplotlib`, `networkx` (via `pip install pydantic-collab[viz]`).
- **Observability:** `logfire` optional extra; `_init_logfire` hooks into configured instances when available.

### Key Libraries
- **`pydantic`** – Validation for `HandOffBase`, settings models, and runtime context data.
- **`pydantic_ai_slim` / `pydantic_ai`** – Agent plumbing, toolsets, and execution contexts.
- **`pytest`, `pytest-asyncio`** – Test framework with async support to simulate networked agents.
- **`ruff`, `pyright`** – Linting and static typing tools configured for strict CI.

### Development Tools
- **`uv`** – Dependency management (`uv.lock`), script runner, and packaging interface.
- **`hatchling`** – Build backend for wheels (see `[tool.hatch.build.targets.wheel]`).
- **`logfire` (optional)** – Structured tracing for agent runs when instrumented.

---

## 🌐 External Dependencies

### Required Services
- **Language Models via pydantic-ai** – The framework defers to whichever providers you configure (`openai`, `anthropic`, etc.). Tests default to `TestModel` so no network calls occur.

### Optional Integrations
- **`logfire`** – Observability; only active if previously configured.
- **Visualization stack (`graphviz`, `matplotlib`, `networkx`)** – Enables `collab.visualize_topology()` to render PNG/SVG graphs.

### Environment Variables
```bash
# Required for demos that use WebSearchTool and similar cloud calls
GOOGLE_API_KEY=<set to a valid key>
```
Store secrets in `.env` (kept out of VCS via `.gitignore`). Never log keys or commit `.env` changes; pass `--env-file .env` to `uv run` when examples need network access.

---

## 🔄 Common Workflows

### Authoring a new topology example
1. Copy an existing script inside `examples/` and rename it with the next sequential prefix.
2. Import the needed Collab class (`Collab`, `StarCollab`, etc.) plus helper tools from `examples/example_tools.py`.
3. Define agents, assign descriptions, and set `agent_calls` / `agent_handoffs` tuples explicitly.
4. Run locally via `uv run --env-file .env examples/<script>.py` and capture the printed execution flow.

**Code path:** `examples/<new>.py` → `pydantic_collab.custom_collabs` → `Collab.run()`

### Validating topology rules after changes
1. Modify orchestration logic in `collab.py` or `_types.py`.
2. Run `uv run pytest tests/test_topology_validation.py -k <focus>` for quick feedback.
3. Execute `uv run pytest` to ensure handoff data and prompt-builder suites still pass.

**Code path:** `pydantic_collab/collab.py` → `tests/test_topology_validation.py`

---

## 📈 Performance & Scale

### Performance Considerations
- `max_handoffs` and `max_agent_call_depth` guard against infinite loops and runaway tool recursion.
- `_allow_parallel_agent_calls` enables parallel tool queries when underlying models support concurrency.
- Usage accounting (`RunUsage`) is copied per `AgentRunSummary` to avoid shared mutable state.

### Monitoring
- Enable `logfire` instrumentation (optional) to trace agent runs; Collab automatically hooks into an already-configured logfire client.

---

## 🚨 Things to Be Careful About

### 🔒 Security Considerations
- Do not commit `.env` or API keys; `.env` currently contains `GOOGLE_API_KEY` for search-enabled demos.
- When sharing examples, scrub execution histories that may include proprietary prompts or thinking sections.
- `CollabSettings` controls what context (conversation, thinking, previous handoffs) is forwarded—review defaults before enabling sensitive data sharing between agents.

*Update to last commit: `d245cd5`*

---
> Source: [Unfold-Security/pydantic-collab](https://github.com/Unfold-Security/pydantic-collab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
