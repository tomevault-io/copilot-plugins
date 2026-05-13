## production-fix-flow

> Version 18.0.0 • Updated 2025-12-23

# PFF Agent Playbook (AGENTS.md)

Version 18.0.0 • Updated 2025-12-23

This playbook is the contract for any coding agent working inside this repo.
If any instruction conflicts with user direction, the user wins.

---

## 0. Prime Directive (read this twice)

1. **Do not change the external behavior** unless the user explicitly asks.
2. **Preserve reproducibility.** Deterministic where possible; document nondeterminism where unavoidable.
3. **Prefer mechanical refactors** over “creative” rewrites.
4. **Architecture-first:** domain/application stay pure; any filesystem/db/network/concurrency/serialization lives in `src/pff/infrastructure/**` (accessed via ports), and cross-cutting helpers live in `src/pff/shared/**`.
5. **No silent “magic.”** No import-time side effects. No hidden global state.
6. **Fail loud, fail early.** Exceptions must be specific; errors must include context.
7. **Performance matters.** Avoid accidental O(N²) and giant in-memory copies. Measure when changing hotspots.
8. **Small diffs by default.** One PR = one intention (except the mechanical flag-day cutover).
9. **Write tests for correctness.** If you fix a bug, add a regression test.
10. **Document contracts.** Docstrings in English; user-facing logs/messages in Portuguese.
11. **Think critically; don't be a yes-machine.** Evaluate every user request on its merits. If an approach is inefficient, fragile, or a better alternative exists, say so with a brief rationale before proceeding. Blind agreement is a bug — reasoned pushback with ROI analysis is expected.

### Shim Exception Policy (Limited)

- **Allowed only for external library incompatibilities** where upstream APIs are removed/changed and no compatible release is available.
- Must be **documented and localized** (single module), and include a clear log entry at `warning` level in English.
- Must be **temporary**: add a TODO with removal criteria (upstream fix or dependency update).
- **Forbidden** for project-owned logic bugs or as a substitute for proper fixes.

## 0.1 Vocabulary (token-efficient)

**Architecture terms (synonyms):** Clean/Hexagonal/Ports&Adapters = domain + ports + adapters.

- `drivers/` = inbound adapters (CLI/API/consumers)
- `infrastructure/` = outbound adapters (DB/FS/HTTP/queues)
- `application/` = use cases + ports
- `domain/` = core logic (pure)

**Principles (use abbrevs in notes):** DRY, SoC, SRP, LoD, KISS, YAGNI.

**Smells (trigger words):**
Duplicated Code; Long Method; Large/God Class; Switch/if-cascade; Primitive Obsession; Shotgun Surgery; Feature Envy; Data Clumps.

**Canonical refactor moves (use these names in plans/PR notes):**
Extract Function/Class; Inline Function/Class; Move Function/Field; Hide Delegate; Remove Middle Man;
Introduce Parameter Object; Replace Conditional with Polymorphism; Decompose Conditional; Guard Clauses.

**Rule:** describe refactors as `1 smell -> 1–3 moves` (max). No essays.

---

## 1. Project overview & architecture

This repo is the production-grade PFF platform focused on:

- DSLFM-KGC training and evaluation
- Probabilistic Circuits (PC2) integration
- Stacking/ensemble orchestration + HPO (Optuna)
- Data quality validation (telecom domain)

Repository map:

- `config/` – YAML specs (tunable knobs live here; code reads configs, not the other way around).
- `data/models/` – Real KG assets (**read-only**; tests must not depend on these).
- `outputs/` – Canonical home for generated artifacts (models, metrics, plots, benches).
- `logs/` – Runtime logs (rotated/dated).
- `src/pff/drivers/` – Composition roots / entrypoints (CLI, API, Celery, HPO).
- `src/pff/application/` – Use cases + ports (orchestration; defines interfaces).
- `src/pff/domain/` – Core business/ML logic (DSLFM-KGC, PC2, anomaly scoring, rules).
- `src/pff/infrastructure/` – Adapters (DB/Postgres, filesystem, queues, external services).
- `src/pff/shared/` – Cross-cutting code used by **2+ production consumers** (strictly curated).
- `scripts/` – Operational scripts (kept thin; long-term they migrate into `src/pff/drivers/` as needed).
- `scripts/lint/` – Unified lint/guardrail pipeline (`lint_repo.py`, `log_lint.py`, `guardrail.py`). Run via `poetry run python scripts/lint/lint_repo.py --fix`.
- `tests/` – Unit/integration/e2e + golden masters + architecture tests.
- `deprecated/` – Legacy modules. Avoid for new code.

### src-layout (packaging hygiene)

- This repo uses `src/` layout (`src/pff/**`). Treat changes here as a top-level structure change: requires codemod + arch tests + editable install workflow.
- Rationale: tests/imports should exercise the installed package, not the working directory copy.

**Rule:** `src/pff/domain/**` and `src/pff/application/**` MUST NOT touch filesystem/network/DB/concurrency primitives. Side effects live in `src/pff/infrastructure/**` and are reached through ports; composition happens in `src/pff/drivers/**`.

---

## 2. Setup & canonical commands

- Install:
  - `poetry install`
- Run lint:
  - `poetry run ruff check .`
- Run formatting:
  - `poetry run ruff format .`
- Run smoke tests:
  - `poetry run pytest -q`
- Smoke suites (examples; pick the smallest relevant):
  - `poetry run pytest tests/audit/test_eval_protocol.py -q`
  - `poetry run pytest tests/data/test_kg_data_quality.py -q`
  - `poetry run pytest tests/database/test_database_schema.py -q`

---

## 3. Mission

You are an agent embedded in a real production R&D pipeline.
Your job is to:

- Ship correct, reproducible improvements.
- Reduce technical debt without breaking behavior.
- Keep HPO/training stable and debuggable.

---

## 4. Non-negotiables (enforced)

### 4.1 Clean-ish Architecture: drivers → application → domain (ports-first)

**Dependency rule (must hold):**

- `src/pff/drivers/**` may import `src/pff/application/**` and `src/pff/infrastructure/**` (composition root).
- `src/pff/application/**` may import `src/pff/domain/**` and defines **ports** under `src/pff/application/ports/**`.
- `src/pff/domain/**` contains models/constraints/scoring; it must not import infrastructure or drivers.
- `src/pff/infrastructure/**` implements ports; it may import domain types, but the dependency arrow must point inward.

**Shared is not a junk drawer:** `src/pff/shared/**` exists only for code used by **≥2 production consumers** (outside tests). If it’s single-use, it belongs in the owning module (`domain/` or `infrastructure/`).
**Test-only helpers:** keep them under `tests/support/**`; do not place test-only utilities in `src/pff/shared/**`.

**Hard rule:** No “quick I/O” in domain/application. If you need data, define a port in application and implement it in infrastructure.

**Flag-day friendly:** avoid creating temporary compat modules or re-export shims. When namespaces change, update imports mechanically (codemod) and delete the old namespace in the cutover PR.

### 4.2 Configuration over hardcoding

- All tunables must come from `config/**` (YAML).
- **Deploy/secrets/env-specific settings:** MUST come from environment variables (env overrides defaults). Never commit secrets.
- No “magic defaults” buried in code without a config knob.
- When adding a new config key:
  - update schema/loader
  - update docs (README or config docs)
  - add a test for config parsing

### 4.3 Output discipline

- All generated artifacts go under `outputs/**`.
- Never write into `data/models/**`.
- Tests must not depend on large real assets.

### 4.4 Determinism contract

- Use explicit seeds for RNG.
- Document nondeterministic ops (GPU kernels, parallelism).
- Avoid time-dependent behavior in core logic.

### 4.5 Logging contract (immutable)

#### 4.5.1. Log level purpose and language

| Level            | Language | Purpose                                                                                                                                       | Examples                                                                                                                      |
| ---------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `logger.info`    | PT-BR    | High-level process steps, user-facing summaries, key metrics at checkpoints, sense of progression, **epoch progress**, **training progress**  | `logger.info("Iniciando treinamento RotatE: epocas=50, entidades=10000")`, `logger.info("Epoca 10/50: loss=0.234, MRR=0.42")` |
| `logger.success` | PT-BR    | Major step completions (use sparingly)                                                                                                        | `logger.success("Treinamento concluido: MRR=0.45, Hits@10=0.82")`                                                             |
| `logger.warning` | EN       | Degraded states, fallbacks, missing optional data                                                                                             | `logger.warning("CUDA unavailable, falling back to CPU")`                                                                     |
| `logger.error`   | EN       | Failures that stop or invalidate the current flow                                                                                             | `logger.error("Training failed: checkpoint corrupted at path=%s", path)`                                                      |
| `logger.debug`   | EN       | Detailed diagnostics, hardware info, shapes, timings, **resource limits**, **internal thresholds**, **adaptive parameters** (debug mode only) | `logger.debug("Batch shape: %s, device: %s", batch.shape, device)`, `logger.debug("Adaptive resource limits: %s", limits)`    |

#### 4.5.2. Structured logging rules

- **Language Policy:** User-facing logs (info/success) MUST be in Portuguese (Brazilian); internal/technical logs (warning/error/debug) and Docstrings MUST be in English.
- **Formatting Policy:** Prefer **f-strings** for readability and modern Python style.
  - ❌ `logger.debug("User {} connected", id)` (Legacy/Lazy)
  - ✅ `logger.debug(f"User {id} connected")` (Modern/f-string)
- **Mandatory Fields:** Every log message must be structured and include:
  - `timestamp`
  - `component_name`
  - `key_parameters` (as a dict or stable keys)
  - `stop_reason` (for early termination or step completion)
- **Noise Control:** Do not claim generic improvements without specific metrics.
  - ❌ `logger.info("Melhoria de 40% na velocidade")`
  - ✅ `logger.info("Tempo medio por epoca: %.3fs -> %.3fs (modelo=RotatE)", old, new)`
- **Observability:** Any change affecting training loops or ensemble logic MUST expose at least one before/after metric and stable keys for run comparison.

### 4.6 No-Shims Policy (cutover contract)

- Do **not** add compatibility layers (`sys.modules` tricks, re-export packages, deprecated aliases).
- If a path changes, the change must be **atomic**: `git mv` + codemod rewrite + delete old namespace.
- Rollback strategy is `git revert` of the cutover commit(s), not “keep shims forever.”

### 4.6.1 No-Wrapper Reexports (token-efficiency)

- Do not keep pass-through modules that only re-export symbols.
- Import from the concrete owner module directly.
- If a wrapper already exists and is not an explicit public API contract, remove it.

### 4.7 Public API & SemVer discipline

- Python import paths are **private by default** unless explicitly listed as public API.
- Public API includes: CLI entrypoints, HTTP routes, artifact formats under `outputs/`, and config schema.
- If public API breaks, bump **major** version and ship a `MIGRATION_GUIDE.md` + codemod when feasible.

### 4.8 No import-time side effects

- Importing a module must not start training, spawn threads/processes, touch DB, or write files.
- Entry points live in `src/pff/drivers/**` (composition root). Modules expose functions/classes; side effects happen only under `if __name__ == "__main__":` or explicit driver calls.

### 4.9 Parquet-Arrow-Postgres-First Policy

This project strictly follows a **tier-based storage architecture**:

1. **Parquet (Archival & Bulk Data):**
    - **Primary format** for tabular data at rest, datasets, and logs.
    - **Use cases:** Large historical datasets, cold storage, batch processing.
    - **Implementation:** `FileManager.read(path, lazy=True)` or `pl.scan_parquet()`.
    - **Optimization:** Use `row_group_size=64k`, `compression=lz4` (default), and `statistics=True`.

2. **Arrow IPC (High-Frequency & Local Cache):**
    - **Primary format** for ephemeral, hot-path data, and inter-process communication.
    - **Use cases:** Local caching (`.cache/`), high-frequency I/O loops, zero-copy reads via `mmap`.
    - **Implementation:** `ArrowIPCHandler` via `FileManager` with `compression="uncompressed"` for local reads.
    - **Rule:** Prefer uncompressed IPC + memory mapping (`mmap=True`) for any data read >10x/minute locally.

3. **PostgreSQL (Operational & Metadata):**
    - **Primary format** for relational state, task queues, HPO trials, and index tracking.
    - **Use cases:** Atomic transactions, structured relations, job status, HPO study history.
    - **Rule:** Never store large binary blobs or giant dataframes directly in Postgres; store the *path* (to Parquet/Arrow) or a summary instead.

**Avoid:** CSV/JSON for intermediate data; convert to one of the above immediately.
**Route all I/O:** Through `src/pff/shared/core/file_manager/handlers/` or the appropriate repository pattern.

### 4.10 Shared-First Policy

Code in `src/pff/shared/**` exists for utilities used by **2+ production consumers**.

**Must route through shared:**

- Filesystem I/O: `FileManager` (not raw `open()`)
- HTTP: `pff.shared.clients.http_client` (not raw `requests`)
- Caching: `CacheManager` (not ad-hoc dicts)
- Concurrency: `ConcurrencyManager` (not raw `threading`)
- Hashing: `stable_hash` (not ad-hoc hashlib)
- Logging: `logger` from `pff.shared`
- Acceleration: `numba_kernels`, `LoopAccelerator`

**Enforcement:** Architecture tests under `tests/architecture/` validate compliance.

### 4.11 Política de Fallback (último recurso)

- Evite fallback como caminho padrão; corrija o caminho principal para cobrir todos os casos esperados.
- Fallback só é aceitável por compatibilidade (ex.: detecção de SO, Ray → Dask).
- Quando inevitável, documente o motivo e garanta equivalência de resultados.

### 4.12 Lifecycle & Interruption (Contract)

- **Cleanup:** Components with persistent state (DB, workers, caches) MUST register cleanup callbacks in `GlobalInterruptManager`.
- **Responsive Loops:** Long-running loops (I/O, training, batching) MUST check `should_stop()` to ensure deterministic and graceful exits.

### 4.13 Comment Discipline (Non-negotiable)

- **Inline Comments:** Avoid/remove unless of extreme value. Focus on *why* (complex logic). Remove non-essential comments to prevent visual clutter.
- **No Self-Documentation:** Never use comments to describe your changes or talk to the user.
- **Language:** Docstrings and technical comments MUST be in English.

---

## 5. Agent execution protocol

### 5.1 Read before you write

- Scan relevant modules, `README.md`, and configs first.
- Identify the “owner layer” (drivers/application/domain/infrastructure/shared).
- Find the nearest existing pattern and follow it.

### 5.2 Propose before refactor (when non-trivial)

- For refactors > ~100 LOC or moving modules, propose:
  - target design
  - expected benefits
  - risk and rollback
  - verification plan
- When touching refactor phases, follow `docs/refactor_planning.md` (v2.1):
  - Fases 0–2 stay in-place (no path changes, no shims).
  - Flag-day cutover uses `git mv` + LibCST codemod (no regex rewrites).

### 5.3 Verification requirement (anti “hallucinated fix”)

Every implementation, refactor, or change MUST include before/after benchmarks and tests to verify performance gains and ensure zero regressions in reproducibility or determinism. **Include at least one test to ensure future breakages are easily identified.**

Every change must include:

- what you changed
- why you changed it
- verification command(s)
- sanity check for refactors: `python -m compileall src scripts tests` (or equivalent)
- expected output / acceptance criteria

### 5.x Agent response format (token-min)

Default reply structure:
A) Decision bullets (<=7 lines) • B) Patch/diff • C) Verify commands.
Use §0.1 vocabulary; do not restate file contents; prefer diffs over full rewrites.

### 5.4 Keep diffs mechanical

- Prefer `git mv` when relocating code.
- Prefer codemods for repetitive import rewrites.

### 5.5 Minimal-diff policy

- Default to small, reviewable diffs.
- Exception: the **flag-day cutover** PR is allowed to be large, but it must be:
  - mechanical (`git mv`, codemod, delete old)
  - behavior-preserving (golden master / characterization tests)
  - reversible (clean `git revert`)

### 5.6 Backward-incompatible changes

- Only with explicit user approval.
- Must include:
  - migration guide
  - deprecation notes (if any)
  - version bump

---

## 6. Data & storage rules

- `data/models/**` is read-only.
- Anything produced by training must go to `outputs/**`.
- DB writes and schema migrations must live in infrastructure and be documented.

---

## 7. Coding conventions

- Python 3.12 features are welcome.
- Avoid `typing` unless needed; `Any` only as a last resort.
- Prefer explicit exception types and high-context error messages.
- Use docstrings in English; keep them short but precise.

---

## 8. Performance & design patterns

- Avoid accidental quadratic loops on KG edges/triples.
- Prefer streaming/iterators over materializing giant lists.
- **Hardware-Aware Scaling:** Use `HardwareManager` (not `os.cpu_count()`) to differentiate physical/logical cores and verify GPU VRAM/compute capability before allocating workers.
- Direct `threading`/`multiprocessing` is permitted **only** in `src/pff/infrastructure/**` or `src/pff/shared/acceleration/**` (and must be documented + tested).
- When optimizing:
  - benchmark before/after
  - pin deterministic settings
  - record results under `outputs/benches/**`

---

## 9. SOTA verification (mandatory web research)

If you propose a new library, method, or refactor approach:

- Read primary docs first.
- Cross-check pitfalls and platform stability (Windows/Linux).
- Summarize the findings and cite the source.

---

## 10. Performance + determinism gates (mandatory)

- Every implementation/refactor must include before/after benchmarks to verify performance gains and ensure zero regressions in determinism or reproducibility.
- **Optimization Workflow:**
  1. **Baseline:** Run benchmark with real data (times/throughput/memory). Ensure baseline tests pass.
  2. **Trial:** Implement candidate optimization.
  3. **Benchmark:** Run benchmark.
  4. **Gate:** **ONLY** integrate/apply changes if gain is proven, all tests are green, and a regression test for the new behavior/performance is included.
  5. **Report:** Provide Before vs After metrics and suggest next optimizations.
- Any change to training loops, losses, batching, masking, or temperature parameters must:
  - include a numeric stability check (NaN/Inf guard)
  - include a deterministic micro-run
  - record stop_reason and metric deltas

---

## 11. Dependency guidance

### 11.1 Layer dependencies (hard rule)

- The dependency arrow points **inward**: drivers → application → domain.
- `src/pff/infrastructure/**` depends on application ports + domain types, never the reverse.
- Dependency injection (DI) lives in the **composition root** (`src/pff/drivers/**`).

### 11.2 Third-party libs

- Before adding/upgrading a third-party library:
  - Read primary docs (or Context7).
  - Summarize key behavior + compatibility risk in PR notes or module docstring.
- Keep `poetry.lock` synchronized with `pyproject.toml`.
- Always record the exact verification command for dependency changes.

### 11.3 Codemods and mass refactors

- For namespace moves or repetitive rewrites: prefer **AST-based codemods** (LibCST-style) over regex when feasible.
- Always support a dry-run mode and add a codemod regression test (input → expected output).
- Verification for a mass move must include:
  - `python -m compileall src scripts tests`
  - grep for old imports (0 hits),
  - architecture tests (`tests/architecture/`),
  - golden master tests (`tests/golden_master/`) when behavior must remain stable.

---

## 12. Testing policy

### 12.1 Test hierarchy

| Level | Type          | Purpose                                                                    | When to run                  |
| ----- | ------------- | -------------------------------------------------------------------------- | ---------------------------- |
| 0     | Static checks | lint/type/format + architecture rules                                      | before commit                |
| 1     | Unit          | pure domain/shared logic; no external services                             | after every change           |
| 2     | Integration   | application + infrastructure with fixtures (DB mocked or ephemeral)        | after business-logic changes |
| 3     | Golden master | characterize CLI/HPO behavior (normalized outputs; CPU-only when possible) | before cutover/release       |
| 4     | End-to-end    | `pff` CLI, API, Celery, full HPO slice                                     | release validation only      |

### 12.2 General rules

- Run fast, relevant tests after every change.
- **Never run the full test suite (`pytest -m "not slow"`) for every change.** The suite is large and slow. Run only the specific test files or directories affected by the change. Reserve full-suite runs for heavy/cross-cutting changes (architecture moves, dependency upgrades, signal handling, shared module edits).
- Never depend on `data/models/**` in tests.
- Prefer deterministic seeds and stable snapshots.
- Every bug fix or feature must add/adjust a regression test to ensure future breakages are easily identified.

---

## 13. Workflow checklist for agents

1. Identify the smallest change that achieves the goal.
2. Locate the owning layer (drivers/application/domain/infrastructure/shared).
3. Confirm config knobs live in `config/**`.
4. Add or update tests (unit/integration/golden master as appropriate).
5. Implement via layers + ports (drivers → application → domain) and put side effects in infrastructure.
6. **Verify:** Run `pytest`, `compileall`, `ruff check .`, `mypy .`, `flake8 .`. Fix ALL static analysis/LSP errors.
7. Write a crisp summary + stop_reason if any early exits.

---

## 14. PR/commit style

- Commit messages must reflect intent.
- If moving files:
  - use `git mv`
  - keep moves separate from logic edits where possible
- Avoid “drive-by” refactors.

---

## 15. Protected areas (extra caution)

| Area | Rule |
| --- | --- |
| `config/**` | Any key change requires docs + config parsing test. |
| `data/models/**` | Read-only. No tests. No writes. |
| `outputs/**` | Only generated content. Never import from here. |
| `src/pff/drivers/**` | Composition root only; keep thin. |
| `src/pff/domain/**` | No side effects. No infra imports. |
| `src/pff/shared/**` + `src/pff/infrastructure/**` core | Must include regression tests under `tests/shared/`, `tests/infrastructure/`, or an explicit golden master. |
| Top-level folder structure | Do not rename without a migration plan + codemod + architecture tests. |

---

## 16. Tools & MCP

- **Documentation search:** Always use `context7` MCP tools to resolve library IDs and retrieve up-to-date documentation for any library, API, or setup step without asking for permission.

---

## References (informative, not exhaustive)

- Semantic Versioning (SemVer)
- Clean Architecture dependency rule
- Composition Root (DI)
- LibCST codemods
- Import Linter architecture tests
- Characterization tests / Golden master testing

---
> Source: [Mentorzx/production-fix-flow](https://github.com/Mentorzx/production-fix-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
