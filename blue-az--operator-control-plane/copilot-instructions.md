## operator-control-plane

> One engine room; PBC above; GT-KB Logbook below; crystals are how narration

# Repository Guidelines

## Where This Repo Sits (read before reasoning about boundaries)

One engine room; PBC above; GT-KB Logbook below; crystals are how narration
arrives; Farm is whose work arrives; LabWired is where hardware evidence comes
from. This CLI is the engine room — the enforcement middle. The **upper boundary**
is PBC (Product Behavior Contracts: what work is allowed). The **lower boundary** is the
GT-KB-derived evidence layer (the Logbook: "do not trust the narration — verify
against the evidence"). `agent-crystallize` crystals and similar session
artifacts are **narration formats arriving at** the lower boundary — untrusted
input in a parseable envelope, never a boundary and never trusted status.
Canonical taxonomy: `project-phoenix/docs/BULKHEAD_TAU_BOUNDARIES.md`. If a spec
or entry in this repo appears to contradict that doc, the doc wins — flag the
drift instead of propagating it.

Cross-repo vocabulary: **BN** = `~/Python/project-phoenix/BOTTLENECKS.md` — the
open-work board that schedules work across BT (Bulkhead Tau, which lives inside
`~/Python/project-phoenix/`) and this repo. BN's header carries the canonical
glossary; BN ≠ BT. Harnesses are **peers** (Claude Code, Codex, Antigravity, Grok,
local lanes — not ranked brands). Frontier seats are expected to **build into the
system**: recover this boundary map from the funnel, calibrate from Phoenix
failure-mode catalogs when working across repos, and leave residue (commits,
catalog entries, handoffs) under the human’s rules — see
`project-phoenix/docs/AGENT_AUDIT_PROTOCOL.md` § “Frontier seat fitness”.

## Concurrent Sessions & Ledger Identity

**Ruled 2026-08-11** (`session-coordination-protocol` task, Q2): when multiple sessions
write to this ledger at once, `--by`/`--assign`/`--review` use the **session-derived id**
(short form of the real Claude Code session id, e.g. `claude-019KSo7K` for
`session_019KSo7KhEUrNJGa1kSVeP8i`), not a role label. Role labels
(`claude-supervisor`, `claude-consultant`, `claude-builder`) were tried first and
drifted twice on this exact ledger — once when two sessions were assumed to be one
(`session-coordination-protocol` handoff-0001), once when a single session's own label
changed three times in 90 minutes. Session ids don't drift by construction; they're
also what the git commit trailers (`Claude-Session: https://claude.ai/code/session_...`)
already carry, which is how the original mixup got resolved in the end.

**Known gap in this ruling, found while implementing it, not resolved by it:** a
session-derived id is not guaranteed stable across a resume. `claude-consultant`'s role
was held by `session_0133KSgM` through 2026-08-09, then by `session_01Hzi1zP` from
2026-08-11 (`3b86ecb`) — same role, same continuity of work, different id. Treat the
table below as tracking **role continuity**, not asserting that an id is permanent.
When a session resumes under a new id and picks up a prior session's thread, add a row
rather than overwrite one, and say so in the handoff that continues the work.

**Recently active sessions** (populate/update as sessions come and go; this is not a
full historical audit — plenty of one-off sessions touched this repo before
2026-08-09 and aren't tracked here):

| Session id | Root / working dir | Recent work | Last active |
|---|---|---|---|
| `claude-01QBpGoE` | `~/operator-control-plane` | Front A infra, `session-coordination-protocol` original proposal (as `claude-supervisor`) | 2026-08-09 |
| `claude-0133KSgM` | `~/operator-control-plane` | Confound pilot passes 1-2, coordination-protocol counter-proposal (as `claude-consultant`) | 2026-08-09 |
| `claude-01Hzi1zP` | `~/operator-control-plane` | `front-e0-desktop-pack-review`, `LOCAL_LANE_CONTRACT_SPEC.md` power-cap fix (continuing `claude-consultant`'s role) | 2026-08-11 |
| `claude-01Q3rn3n` | `~/operator-control-plane` | Front G, pa-evidence Gate 1 adapter | 2026-08-10 |
| `claude-019KSo7K` | `~/Alignerr` (cross-repo, via SSH to desktop) | Front D dashboard verification, Front E0 desktop pack, Q1/Q4 ruling implementation | 2026-08-11 |
| `claude-01FSgUqu` | `~/.dotfiles` on **desktop** (cross-repo: this repo + `~/Python`) | Runner trace retention (`--trace-dir`), E1 27-cell desktop matrix + `FINDING.md`, gemma4:31b throughput correction across GOLD_STANDARD/Phoenix docs, SEAT-COST-001 cross-check | 2026-08-12 |

Registering a session-derived id as a harness (`.operator/harnesses/claude-<id>.yaml`)
is required only if something needs to `--assign`/`--review` to it — `--by` needs no
registration (Q1 ruling, same task). `.operator/` is gitignored and per-machine, so a
harness registered on z13 does not exist on desktop until copied there.

## Project Structure & Module Organization

This repository is a compact Python CLI project (requires Python ≥ 3.12).

| Path | Role |
|------|------|
| `operator` | Main ledger CLI (~6000 lines; most repo-local changes land here) |
| `opr` | Confirmation-gated governed REPL for local-model sessions |
| `operator-broker` / `authority_broker.py` | Standalone P3a authority broker |
| `operator-admin` / `authority_admin.py` | Root-managed P3b policy install/lifecycle |
| `authority_client.py` / `authority_projection.py` | Enrolled CLI ↔ broker integration |
| `socket_permission_helper.py` | Socket permission helpers for broker paths |
| `dogfood_runner.py` | Typed, resumable dogfood plan/run engine (issue #8), invoked via `operator-admin dogfood-*` |
| `tests/test_operator.py` | Repo CLI integration suite (temp workspaces) |
| `tests/test_opr.py` | Governed REPL coverage |
| `tests/test_authority_*.py` | Broker, admin, integration, upgrade suites |
| `tests/fixtures/` | Synthetic harness logs and pricing fixtures |
| `*_SPEC.md` | Product/behavior contracts (source of truth for semantics) |
| `OPERATIONS_RUNBOOK.md` | Operator recovery and P3 procedures |
| `owners-manual/` | User-facing manual, chapters, PBC drafts, figures |
| `.operator/` | **Local runtime ledger only** — gitignored; never commit |

The P3a broker must remain independent of the repo-local `operator` CLI and
`.operator` state. P3b owns only root-managed installation and policy lifecycle.
`CRYSTAL_LEDGER_INTEROP_SPEC.md` is **implemented, Phases 1–3 (2026-07-18)**:
`crystal-attach`, `crystal-import` and `crystal-bridge` are all registered and
live. Its trust rules are enforced in code, not aspirational — `crystal-attach`
fails closed if a caller smuggles a status onto the namespace (T2; verification
goes through `evidence-attach --status` as a distinct verifier). Read the spec's
own Status line for phase state rather than trusting a summary here.

## Build, Test, and Development Commands

- `pip install -r requirements.txt` installs the only runtime dependency, PyYAML.
- `./operator --help` lists the 22 repo CLI subcommands.
- `./operator doctor` checks consistency of the local `.operator/` ledger.
- `pytest tests/` runs the full subprocess-driven integration suite.
- `pytest tests/test_operator.py -q` is the fastest focused repo-CLI command.
- `pytest tests/test_authority_broker.py -q` runs the standalone broker/store suite.
- `pytest tests/test_authority_admin.py -q` runs the P3b install/policy suite.
- `pytest tests/test_authority_integration.py -q` runs CLI ↔ broker integration tests.
- `./operator-broker --help` lists the isolated P3a development surfaces.
- `./operator-admin --help` lists owner-only P3b commands. Production use requires a
  root-owned staged or installed copy; do not run it through sudo from this checkout.
- Lint/format (from `pyproject.toml`): `ruff check .`, `black --check .`, `isort --check-only .`.

Run `./operator init` only in a throwaway or intended workspace; it creates local
ledger files under `.operator/`. Re-running on an existing YAML-only ledger baselines
those records into SQLite without changing visible IDs.

## Coding Style & Naming Conventions

Use Python 3 with 4-space indentation, `from __future__ import annotations`, and
standard-library modules before third-party imports. Existing code favors small
helper functions, explicit paths via `pathlib.Path` or `os.path`, and readable CLI
output over framework abstractions. Keep record IDs in the established sequential
forms: `claim-0001`, `evidence-0001`, `usage-0001`, `handoff-0001`. CLI subcommands
and flags use kebab case, for example `task-create`, `--verified-by`, and
`task-transition`.

## Testing Guidelines

Tests use `unittest` assertions under pytest. Add repo CLI coverage in
`tests/test_operator.py` for CLI behavior, file layout, YAML contents, exit codes,
and stdout/stderr messages. Use temporary directories for ledger mutations, following
the existing `setUp` and `tearDown` pattern.

- Broker protocol, kernel-credential, transaction, CAS, and recovery coverage belongs
  in `tests/test_authority_broker.py`; it must not require sudo or simulated socket credentials.
- Administrative path, privilege-drop, policy lifecycle, preflight, and crash recovery
  coverage belongs in `tests/test_authority_admin.py`.
- Enrolled CLI transition/reconcile coverage belongs in `tests/test_authority_integration.py`.
- Put reusable synthetic logs or manifests in `tests/fixtures/`.
- Keep issue #6 repo CLI integration and issue #7 real-host dogfood out of P3b unit work
  unless the task explicitly targets them.

Test identity hooks (preserve both halves of the spoof guard):

- `OPERATOR_TEST_UID` + `OPERATOR_TEST_SENTINEL` (`1`/`true`) override the executing UID.
  UID without sentinel is a spoof attempt that `doctor` must flag as Error.
- `OPERATOR_TEST_CLAUDE_DIR` / `OPERATOR_TEST_CODEX_DIR` / `OPERATOR_TEST_GEMINI_DIR`
  redirect usage import to fixtures.
- `OPERATOR_MACHINE` overrides the `executor.machine` provenance stamp
  (see `MACHINE_PROVENANCE_SPEC.md`).

## Commit & Pull Request Guidelines

Recent commits use concise imperative subjects, such as `Add doctor checks...` or
`Refine worked example...`. Keep commits scoped to one behavioral or documentation
change. Pull requests should describe the user-visible change, list verification
commands such as `pytest tests/`, and call out ledger, identity, verification, or
authority-broker semantics that changed. Include screenshots only for rendered manual
or diagram updates.

## Security & Configuration Tips

Identity and verification behavior is governed by `.operator/identity.yaml`.
Preserve fail-closed verification semantics: only an enforced, registered verifier
OS UID distinct from the claim author is `uid_isolated`; `single_user` verification
is advisory. Evidence should prefer re-runnable commands over static blobs, but the
operator must never execute stored verification commands. In P3 mode, use
`task-transition` / `authority-reconcile` for broker-backed verified/complete state;
do not smuggle those statuses through `session-end`.

## Lane Economics

Cost is the driver's price, not the author's (see `usage-summary --by-lane`).
Transcript chores — summaries, handoff briefs, session rehydration — are
cheap-lane work and should never run in the most expensive seat at the table.
If a long-session resume or compaction is about to bill to a scarce frontier
seat, warn the operator before it starts.

Harness names are not ranks. Any agent that works under the ledger's identity,
claim, evidence, and verification rules is a participant; `assigned_harness`,
`review_harness`, `harness_id`, and `lane` are routing, provenance, and economics
fields, not a caste system. Do not infer supervisor, builder, verifier, or lane
authority from brand name alone.

---
> Source: [blue-az/operator-control-plane](https://github.com/blue-az/operator-control-plane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
