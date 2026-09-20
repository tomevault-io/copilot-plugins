## agnara-execution-runtime

> > **Audience:** Claude Code, OpenAI Codex, Antigravity, and other autonomous AI coding agents inspecting, maintaining, or extending this codebase.

# AGENTS.md — Operational Manual for Autonomous AI Agents

> **Audience:** Claude Code, OpenAI Codex, Antigravity, and other autonomous AI coding agents inspecting, maintaining, or extending this codebase.
> **Role:** Single source of operational truth for machine agents. For human-oriented documentation, see `README.md`.

---

## 1. Project Identity & Historical Context

- **Repository:** `agnara-project/agnara-execution-runtime`
- **Designation:** **Agnara Historical Reference Application #005**
- **Framework Version:** Strictly pinned to **`agnara==0.1.0a3`**
- **Python Version:** **CPython >= 3.14** (designed for free-threaded compatibility under PEP 703)
- **Status:** **Historical / Frozen**
- **Mission:** Explain what happens inside Agnara after declaring a capability, using an order processing flow (`create_order`) to visualize the runtime execution cycle: `create_order -> ExecutionPlan -> Invocation -> ExecutionContext -> Result`.

---

## 2. Inviolable Architectural Constraints

1. **Do Not Upgrade Agnara:** Under no circumstances should `pyproject.toml` or `requirements.txt` be altered to reference versions later than `0.1.0a3` or unreleased development branches (`main`/`develop`).
2. **Do Not Invent Speculative APIs:** Only use the actual public API surface present in `agnara==0.1.0a3`. Do not assume `Agnara` owns execution or DI containers (those belong to `agnara.execution` and `agnara.core.di`).
3. **No External Infrastructure:** Do not introduce web frameworks (FastAPI, Flask), external database drivers, or unnecessary network mocks. Domain models and services remain in-memory Python structures.
4. **Preserve CPython 3.14+ Compatibility:** Maintain strict compatibility with modern Python 3.14+ idioms, including free-threaded execution semantics (lock-free reads on frozen objects).
5. **Separation of Authoring and Execution:** `@app.capability` records declarations on the authoring surface; execution must be performed through compiled `ExecutionPlan` and `ExecutionContext`.

---

## 3. Codebase Structure & Ownership

```
agnara-execution-runtime/
├── .agents/skills/             # Agent-first specialized workflows
│   ├── agnara-runtime/SKILL.md # Runtime compilation and execution workflow
│   ├── documentation/SKILL.md  # Documentation sync and style invariants
│   └── testing/SKILL.md        # Quality gates and test invariant verification
├── .github/                    # GitHub repository automation & templates
│   ├── ISSUE_TEMPLATE/         # Bug report and doc improvement issue forms
│   ├── PULL_REQUEST_TEMPLATE.md# Pull request validation checklist
│   ├── dependabot.yml          # Dependabot configuration ignoring agnara
│   └── workflows/ci.yml        # CI workflow (Python 3.14 on Ubuntu & Windows)
├── docs/                       # Architectural and technical documentation
│   ├── execution-lifecycle.md  # Detailed 6-stage execution lifecycle specification
│   └── public-api-boundary.md  # Permitted public APIs vs internal boundaries
├── AGENTS.md                   # Operational instructions for AI agents (this file)
├── ARCHITECTURE.md             # Deep architectural breakdown of execution pipeline & DI lifecycle
├── README.md                   # Human-oriented progressive learning guide
├── CHANGELOG.md                # Historical release ledger
├── CONTRIBUTING.md             # Contribution rules under Historical/Frozen status
├── LICENSE                     # Apache 2.0 License
├── SECURITY.md                 # Security reporting policy
├── pyproject.toml              # Hatchling build configuration & dev dependencies
├── requirements.txt            # Exact pinned core dependency (agnara==0.1.0a3)
├── domain.py                   # Pure domain models (Order, InventoryItem) & domain exceptions
├── services.py                 # In-memory services (InventoryService, AuditSession) & DI providers
├── orders.py                   # Agnara capability declarations & execution plan compiler
├── app.py                      # Interactive CLI runner executing the 7 runtime phases
└── tests/
    ├── __init__.py
    ├── test_compilation.py             # Plan compilation, signature analysis, parameter protection
    ├── test_context_and_invocation.py  # Invocation & ExecutionContext invariants and deadline math
    ├── test_di_resolution.py          # Scoped resolution (Singleton vs Invocation) & generator teardown
    ├── test_execution_outcomes.py      # invoke vs invoke_result, canonical error mapping, redaction
    └── test_telemetry.py               # TelemetryHook lifecycle, nanosecond timing, exception isolation
```

---

## 4. Reproducible Command Palette

Agents performing modifications, health checks, or code reviews must run commands using the local virtual environment:

```powershell
# 1. Clean Environment Initialization
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1

# 2. Dependency Installation
pip install -r requirements.txt
pip install -e ".[dev]"

# 3. Interactive CLI Demonstration
python app.py

# 4. Comprehensive Test Suite
pytest -v

# 5. Code Style & Formatting Verification
ruff check .
ruff format --check .

# 6. Packaging Verification
pip wheel . --no-deps -w dist
Remove-Item -Recurse -Force dist
```

All verification commands (`pytest -v`, `ruff check .`, `ruff format --check .`, `python app.py`, `pip wheel`) must exit with return code `0`.

---

## 5. Public API Surface in `agnara==0.1.0a3`

When reading or authoring code, use only these verified public exports:

| Symbol | Module / Origin | Purpose in Runtime Execution |
|---|---|---|
| `Agnara(name: str)` | `agnara.application` | Authoring root owning `CapabilityRegistry`. Does not execute. |
| `app.compile()` | `agnara.application` | Freezes registry into `FrozenCapabilityRegistry` (ADR 0005). |
| `CapabilityId` | `agnara.capability.identity` | Qualified identifier (`namespace.name`). Immutable dataclass. |
| `ExecutionPlan` | `agnara.execution.plan` | Compiled execution plan. Factory: `ExecutionPlan.compile(definition, registry, ...)`. |
| `Invocation` | `agnara.execution.invocation` | Protocol-neutral request (`capability_id`, `payload`, `metadata`, `deadline`). |
| `ExecutionContext` | `agnara.execution.context` | Per-invocation execution environment (`invocation`, `di_container`, `tracking_id`, `state`). |
| `invoke` | `agnara.execution.runtime` | Ergonomic in-process execution. Propagates raw Python values and exceptions. |
| `invoke_result` | `agnara.execution.runtime` | Safe execution returning `CanonicalResult[T]` (`Success[T]` or `Failure`). Redacts internal errors. |
| `Success`, `Failure`, `FailureCode` | `agnara.execution.result` | Canonical execution outcomes and standardized failure categories. |
| `TelemetryHook` | `agnara.execution.telemetry` | Protocol for synchronous observers (`on_invocation_start`, `on_invocation_terminal`). |
| `DIRegistry`, `DIContainer` | `agnara.core.di` | Dependency injection registry and runtime container managing singletons and invocation scopes. |
| `@provider(scope=...)` | `agnara.core.di.provider` | Decorator defining scoped dependency providers (`Scope.SINGLETON`, `Scope.INVOCATION`). |

---
> Source: [agnara-project/agnara-execution-runtime](https://github.com/agnara-project/agnara-execution-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
