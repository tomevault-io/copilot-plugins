## jacobian

> Follow [CONTRIBUTING.md](CONTRIBUTING.md) for setup, validation, documentation,

# Repository Guidelines

Follow [CONTRIBUTING.md](CONTRIBUTING.md) for setup, validation, documentation,
commits, and pull requests. This file lists only Jacobian-specific constraints.
Load the [product model](docs/explanation/product-blueprint.md),
[goals](docs/explanation/goals.md), or
[tool reference](docs/reference/tools.md) when needed. For built-in mathematical
operations, also use the
[domain operation library reference](docs/reference/domain-operation-library.md).

## Product Constraints

Jacobian gives agents composable mathematical capabilities. Its principles are:

- a broad mathematical portfolio;
- atomic, agent-visible outcomes;
- agent-owned composition and research strategy;
- inspectable intermediate artifacts; and
- independent verification of exact claims and evidence.

Each capability exposes one coherent, inspectable mathematical outcome. It may
coordinate backend calls, but useful intermediate artifacts, failures,
relationships, scope, completeness, assurance, and proof obligations remain
visible.

Jacobian exposes mathematical affordances, not research policy. Capabilities
must remain atomic, searchable, and freely composable. Do not prescribe
preferred decompositions, proof strategies, cross-capability workflows,
verification order, or stopping criteria through discovery, ranking, prompts,
or adapters. Capabilities may implement specific mathematical methods, while
agents remain free to choose and compose them. Prompts and resources may
explain protocol and evidence semantics, but remain optional.

Design against the existing portfolio. Reuse typed artifacts that expose the
needed outcome; declare overlap and keep useful intermediates. Before
stabilizing or recommending a capability, inspect nearby catalog entries by
domain, artifact type, and outcome. If overlap remains ambiguous or
consequential, use the
[evaluation plan](docs/reference/capability-workflow-evaluations.md). Routine
additions need no exhaustive pairwise or leave-one-out evaluation.

Use `capability.describe(query=...)` for intent-led search,
`capability.describe(capability_id=...)` for exact contracts and invocation
examples, and `capability.invoke` to execute. Use `capability://catalog` for the
complete machine-readable inventory. Prefer domain-owned capability IDs to
generic schemas, verb taxonomies, mechanical backend wrappers, or new top-level
MCP tools.

Prefer thin adapters to maintained mathematical systems. Pin versions when
reproducibility, certificates, or verification depend on them.

Built-in mathematical producers belong in explicit domain bundles. Do not add
global operation registries, recursive package discovery, import-time
registration, or mechanical wrappers for backend functions. Producers remain
capped at `COMPUTED`; domain-owned checker declarations do not authorize
themselves.

Keep availability, recommendations, compatibility, and verification authority
separate. Experimental contracts may break between versions; compatibility
applies only to supported versions. Only an operator-authorized checker
independent of proposal, search, and evaluation may return `VERIFIED`.

Follow the
[ownership model](docs/explanation/product-blueprint.md#ownership-model).
Keep strategy out of the kernel, semantics out of generic contracts, and
checker authorization out of plugins and search code.

## Fail-Closed Verification Rules

- Treat `TIMEOUT`, `CANCELLED`, `ERROR`, incomplete enumeration, and failure to
  find a witness as non-conclusions.
- Never promote an evaluator score, solver status, model answer, or search
  result directly to `VERIFIED`.
- Keep execution status, input validity, mathematical conclusion, assurance,
  and evidence type separate.
- Bind `VERIFIED` evidence to the exact claim, semantics, candidate, scope,
  certificate format, and checker identity.
- Plugins and search code cannot authorize checkers or change trust policy.
- Independent checkers cannot depend on the search implementation they certify.

## Repository Gotchas

- Before final validation, use `make test-plan BASE=<revision>` and run the
  selected gate on the final tree. In a shared checkout, agents must own
  disjoint paths and must not switch branches, stage, commit, clean, or rewrite
  shared files until their work is integrated.
- Jacobian is pre-stable. Release specifications capture supported snapshots;
  they do not order capability research.
- Validate the complete Pydantic request model before computation or artifact
  writes. JSON Schema supports discovery; it does not replace cross-field
  validation.
- `COMPLETED` bounded execution may still have `UNKNOWN` completeness and open
  obligations. Execution completion does not establish optimality or a
  mathematical conclusion.
- Include every first-class artifact reference, including verification records,
  in the result's `artifact_uris`.
- An unavailable optional provider must remove only the affected capabilities;
  unrelated kernel startup and catalog entries remain available.
- Keep `deep_review.md` local; it is ignored and is not design source material.
- Keep worked cases in reference scenarios and benchmarks.

## Agent Workflow Entry Points

Repository-local skills under `.agents/skills/` are the maintained entry points
for recurring capability work:

- use `develop-math-capabilities` for the complete challenge-to-evaluation
  improvement loop;
- use `discover-math-capabilities` to mine recurring mathematical moves and
  produce evidence-gated candidates;
- use `implement-math-capability` for a bounded producer contract;
- use `implement-math-capability-checker` for an operator-authorized,
  independent verification path; and
- use `evaluate-math-capabilities` for frozen public reproductions or held-out
  comparative evaluations.

Read the selected skill completely and use
[capability development handoffs](docs/reference/capability-development-handoffs.md)
between stages. Record the git tree, installed catalog and policy digests,
provider/runtime state, model and prompt settings, raw trace location,
validation actually run, unresolved obligations, and next action. Do not use
ignored transcripts or local review notes as the only handoff.

For remote MCP operation, use
[Deploy the remote MCP server](docs/how-to/deploy-remote-mcp.md) and the
checked-in files under `deploy/`. They define the reproducible systemd, Caddy,
Tailscale Funnel, smoke, restart, and rollback baseline. Files under `tmp/` are
ignored host-local evidence and are never deployment source of truth. Compare
the MCP-advertised package version with the selected checkout during every
redeploy; an unchanged catalog does not prove that the backend restarted.

## Cursor Cloud specific instructions

This is a Python 3.12 project managed with `uv`; the base image ships Python 3.12
and Node but not `uv`. The startup update script installs `uv` (to
`~/.local/bin`, added to `PATH` via `.bashrc`/`.profile`) and runs
`uv sync --locked --dev`, so dependencies (including the `flint`/`smt` dev-group
backends `python-flint`, `cvc5`, `z3-solver`) are already installed when a
session starts. Standard dev, test, lint, and build commands live in the
`Makefile` (`make help`) and `CONTRIBUTING.md`; use those rather than duplicating
them.

Non-obvious caveats:

- If a fresh non-login shell can't find `uv`, run `export PATH="$HOME/.local/bin:$PATH"`.
- Optional backends are absent by default and their capabilities are correctly
  omitted: `lean.check` prints `lean.check is not installed` on `init`/startup
  (the pinned Lean 4.31.0 toolchain is not installed), and external solver
  executables (`cadical`, `drat-trim`, `carcara`) are not on `PATH`. This does
  not break the kernel, catalog, or the core test suites. Only install Lean/elan
  or those executables when specifically exercising `lean_runtime` tests or SAT
  proof-artifact capabilities.
- `make test-unit` is the quick unit lane and `make check` combines it with lint
  and typecheck. Use `make test-all-ci` only for an explicit exhaustive local
  reproduction. Never run bare `uv run pytest` across the whole suite — it mixes
  provider and Lean boundary tests into one pool; use a focused `make test-*`
  target instead.
- Only the coordinating agent may start an exhaustive test lane. Never delegate
  one to a parallel agent sharing the host. Before an exceptional broad run,
  inspect active processes for pytest jobs from this checkout and stop or wait
  for them; concurrent runtime/store/subprocess suites turn the 60-second test
  timeout into a host-contention detector rather than useful failure evidence.
- SQLite is one visible contention point, but not the sole cause: full-runtime
  construction also performs durable filesystem publication, subprocess
  startup, schema registration, and CPU-heavy capability setup. A timeout
  observed in `PRAGMA`, `fsync`, `os.link`, or process startup under concurrent
  suites must be reproduced with the owning focused test before it is treated
  as a product defect.
- Quick end-to-end smoke of the product surface: `uv run jacobian --state-dir .jacobian init`
  (CLI), or start the MCP server with
  `uv run jacobian-mcp --transport streamable-http --host 127.0.0.1 --port 8000 --allow-anonymous`
  (remote transports require `--allow-anonymous` or `--auth-tokens-file`; stdio is
  the default transport). The runnable
  `docs/tutorials/first-verified-result.md` script exercises the full
  find-then-independently-verify flow.

---
> Source: [morluto/jacobian](https://github.com/morluto/jacobian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
