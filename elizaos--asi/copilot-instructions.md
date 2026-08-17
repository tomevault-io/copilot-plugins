## asi

> JAX continual-learning research track for elizaOS

# Alberta Framework — agent guide

JAX continual-learning research track for elizaOS
([The Alberta Plan](https://arxiv.org/abs/2208.11173)). This tree is a
**development fork** of `lalalune/alberta` (fork point `2ac3533`), not a
lightly-patched vendor copy — see `VENDORING.md` for the divergence summary
and the canonical upstream URL. The robot track imports the continual-RL
subset in-process; keep `requires-python >= 3.12` and the `numpy >= 1.26`
floor intact.

**Current headline lane:** the IPMNIST screening/confirmation campaign,
which is development-grade and permanently nonpromoting. Results move; read
the `summary_*.json` files and `publication_runs/RESULTS.md` under
`outputs/ipmnist_screening/` instead of copying numbers into overview docs,
and re-measure the selected baseline before any A/B. The theory snapshot is
`docs/research/ipmnist-theory.md`; raw records and audits live beside the
outputs. Check `docs/evidence/negative-results.md` before retrying an idea.

## Layout

```
alberta_framework/
  core/        learners and adaptive optimizers, Horde, prediction/control,
               learned state, memory, world models, dreaming,
               options/STOMP/OaK, feature lifecycles, and PrototypeAgent
  streams/     synthetic prediction streams + gauntlet, closed_loop,
               pavlovian, recurring_multiagent
  evaluation/  strict evidence artifacts, validators, the evidence registry,
               evidence CLIs, and bounded development diagnostics
  benchmarks/  IPMNIST lanes (upgd_ipmnist, ipmnist_screening,
               upgd_label_emnist), Forager integrations
  utils/       multi-seed experiments, statistics, metrics, export
  steps/       public Step 1–12 kernels: smoke CLIs for Steps 1–2,
               pipeline.py consumes Steps 3–4, Steps 5–12 are
               library-surface only (cited by docs/status.md)
outputs/       evidence + campaign artifacts — see immutability rules below
tests/         unit, integration, scientific, and replay checks
```

Key documents:

- Status & evidence: `docs/status.md` (levels L0–L3, completion gates) ·
  `docs/evidence/methodology.md` (property-by-property map)
- Active campaign: `docs/research/ipmnist-theory.md` ·
  `outputs/ipmnist_screening/{RUNBOOK,FINAL_REPORT,AUDIT,CEILING_ANALYSIS,SOTA_LANDSCAPE_2026}.md`
- Durable records: `docs/archive/forager-comparator-audit.md` ·
  `docs/design/rtu-taylor-correction.md` ·
  `docs/evidence/negative-results.md`
- Runbook: `docs/runbooks/foragax-open-screen.md`
- Benchmarks/infra: `FORAGER_BENCHMARK.md` ·
  `docs/archive/historical-forager-reconstruction.md` · `VENDORING.md` · `CHANGELOG.md`

`README.md` is the index; if you add a root doc, link it there.

## Running things

Always use the project venv:

```bash
.venv/bin/python -m pytest tests/<file> -q                  # one file
.venv/bin/python -m pytest tests -q                         # full suite
.venv/bin/python -m pytest --collect-only -q | tail -1     # count of record
.venv/bin/python -m ruff check .                           # lint (line length 100)
.venv/bin/python -m mypy                                   # strict, py312
.venv/bin/alberta-evidence-status                          # evidence registry
```

See `[project.scripts]` in `pyproject.toml` for the current console-script
inventory. The ones you'll reach for are `alberta-evidence-status`,
`alberta-forager-benchmark`, `alberta-foragax-open-screen`, and the
`alberta-forager-matched-*` family. Benchmark executions happen through
scripts/CLIs, never inside pytest — tests must stay CI-cheap unless
explicitly registered as a scientific lane.

## Marker lanes

- `unit` — fast isolated behavior/contract tests; never scientific evidence.
- `integration` — spans components, persistence, or process/CLI boundaries.
- `scientific` — frozen promoted-evidence protocols; may be expensive and
  require preregistered seeds.
- `slow` — wall-clock heavy modules (>~30s serial); excluded from the fast
  per-PR CI lane.
- `package` — built-distribution and installed-entry-point contracts; isolated
  in the package CI lane.

The fast runtime selector is `-m "not slow and not package"`.

## Evidence-promotion rules (fail-closed)

- **Never auto-promote.** Passing tests, replays, or reruns do not upgrade a
  claim. Promotion requires a frozen preregistered protocol, untouched
  held-out seeds, a versioned artifact schema, and its strict validator
  accepting the artifact.
- **Frozen seeds stay frozen.** Calibration/development seeds and consumed
  evidence seeds can never be reused for promotion. Consumed-seed replays are
  explicitly nonpromoting.
- **Pinned `outputs/` artifacts are immutable.** Never overwrite, edit, or
  delete `outputs/ftl_decision/` (sha-pinned), `outputs/continual_ia/`
  (historical chain + source snapshot), `outputs/recurring_feature/`,
  `outputs/scale_robust_feature/evidence.v2.json`,
  `outputs/continual_multiagent/`, `outputs/step2_canonical/`,
  `outputs/evidence_manifest.json`, the sealed/`QUARANTINED.md` forager
  roots, or the chmod-frozen negative-result dirs. New runs write to NEW
  paths and new schema versions. `outputs/ipmnist_screening/` and
  `outputs/upgd_ipmnist/` hold the active campaign's development artifacts —
  append, don't rewrite.
- **Registered source hashes are load-bearing.** Editing a registered source
  file invalidates persisted evidence until the frozen protocol is rerun; the
  registry reports `invalid` (exit 2) — that is working-as-designed, not a
  bug to silence. Read `evaluation/evidence_manifest.py` for the current
  five-claim source inventory, and inspect each development lane's own source
  manifest before touching it. Counts in narrative docs are not authoritative.
- Thresholds are calibrated empirically on development data with ≥2x margins,
  then frozen. Retuning a threshold after seeing held-out results is
  disallowed (a failed gate stays a valid rejection).
- Library changes are failing-test-first; state is frozen chex dataclasses;
  RNG uses explicit `jr.key(...)` seeds.

## Evidence registry (5 claims)

`alberta-evidence-status` exits `0` (all accepted), `1` (valid rejection or
missing), `2` (invalid). Each claim's CLI is also
`.venv/bin/python -m alberta_framework.evaluation.<module>`.

Run the command for live status. A claim becomes `invalid` when its registered
source bytes no longer match the pinned artifact; unrelated dirty-worktree
changes alone are not a registered-source mismatch. The frozen outcomes
recorded in the pinned artifacts are:

| Claim | Frozen artifact outcome | Artifact | CLI |
|---|---|---|---|
| `recurring_pair_features` | accepted (narrow L2) | `outputs/recurring_feature/evidence.v1.json` | `alberta-recurring-feature-evidence` |
| `scale_robust_pair_features` | accepted (narrow L2) | `outputs/scale_robust_feature/evidence.v2.json` | `alberta-scale-robust-evidence` |
| `ftl_world_model_decision_fidelity` | accepted (historical chain) | `outputs/ftl_decision/evidence.v1.json` | `alberta-ftl-evidence` |
| `recurring_multiagent_coadaptation` | accepted (narrow L2) | `outputs/continual_multiagent/evidence.json` | `alberta-multiagent-evidence` |
| `continual_intelligence_amplification` | valid rejection (frozen 10% gate) | `outputs/continual_ia/evidence.json` | `alberta-ia-evidence` |

No accepted claim is an Alberta Plan completion; keep README/status wording
narrow and honest.

## Files that are load-bearing outside the docs

- `FORAGER_BENCHMARK.md` is hashed into Forager run provenance
  (`forager_cli._source_tree_sha256`) — edits change benchmark receipts.
- `README.md`, `CHANGELOG.md`, and `FORAGER_BENCHMARK.md` ship in the sdist.
- The CHANGELOG version heading is asserted by `test_release_metadata.py`.
- The robot track imports `core/{actor_critic,continual_backprop,
  initializers,normalizers,optimizers,sarsa}` via `import
  alberta_framework` — `alberta_framework/__init__.py` must stay importable,
  so every module deletion is a two-file change.

## Conventions

- ruff line length 100; ESM/TS conventions do not apply here — this is a pure
  Python track.
- `CLAUDE.md` and `AGENTS.md` are identical: author `CLAUDE.md`, copy to
  `AGENTS.md`.
- No git commits unless explicitly asked.

---
> Source: [elizaOS/asi](https://github.com/elizaOS/asi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
