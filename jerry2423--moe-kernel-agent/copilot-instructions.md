## moe-kernel-agent

> > This file is always loaded into context. Everything here supersedes paraphrases elsewhere. If a phase file contradicts this, this wins — fix the phase file.

# Harness — Operator's Manual for Claude

> This file is always loaded into context. Everything here supersedes paraphrases elsewhere. If a phase file contradicts this, this wins — fix the phase file.

## What this repo is

A three-phase harness for producing a high-performance Triton kernel that matches a reference PyTorch implementation. Orchestration lives in `TASK.md`; anti-cheat invariants live in `RULES.md`; Phase-3 performance guidance lives in `HINTS.md` (do **not** read it before Phase 3).

## Directory layout (authoritative)

```
TASK.md RULES.md HINTS.md HANDOFF.md ITERATIONS.md   Top-level contracts & state
CLAUDE.md                                            This file
input/                                               Reference PyTorch (Phase 2 input). Read-only.
context/                                             Curated reference material (Phase 1 input). Read-only.
  INDEX.md                                           Bottleneck → technique → source map. Seed for knowledge/INDEX.md.
  schemas/*.schema.json                              Draft-07 JSON Schemas for every structured artifact.
  moe_workloads.jsonl sota_targets.jsonl ds_moe.json Workload & target definitions.
knowledge/                                           Phase 1 output.
solution/                                            Phase 2 + 3 output.
  fused_reference.py  fusion_notes.md                Stage 2.1
  subgraph_specs.json                                Stage 2.2
  triton/kernel.py                                   Stage 2.3 / 2.5 / 3.C
  rules_check.md  rules_check.json                   Stage 2.4 (markdown + machine-readable)
  autotune_notes.md                                  Stage 2.5
  workload_classes.json                              Phase 3.A
  iterations.jsonl                                   Phase 3.B (append-only, row-per-iteration)
bench/                                               Modal-driven benchmark/profile/autotune scripts. Read-only except per bench/GUIDE.md update rules.
tools/                                               Harness-level Python utilities (validators). Committed.
scripts/                                             Project-generated helpers (bench.sh). Gitignored.
trajectory/                                          Per-iteration captured artifacts. Gitignored.
.claude/
  commands/                                          Phase slash-commands.
  hooks/                                             PreToolUse / SessionStart / Stop hooks.
  settings.json                                      Committed allowlist + deny + hooks.
  settings.local.json                                User overrides (gitignored by default).
  state/                                             Per-session scratch (current_phase etc). Gitignored.
  skills/                                            Reusable capability docs.
```

## Environment (B200 via Modal, no local GPU)

- Pre-built conda env `fi-bench` has `modal`, `flashinfer-bench`, all deps. Do **not** `pip install` or create a new env.
- Because shell state does not persist between Bash tool calls, any direct `modal` invocation must be prefixed **in the same command** with:
  ```bash
  eval "$(conda shell.bash hook)" && conda activate fi-bench && <your command>
  ```
  `scripts/bench.sh` handles this for you. A `PreToolUse` hook rejects `modal run` without this prefix.
- Always pass `-m` to `modal run`: `modal run -m bench.modal_<name>`. File-path syntax (`modal run bench/modal_<name>.py`) causes import failures. A `PreToolUse` hook rejects the file-path form.
- Kernel entry point: `run()` in `solution/triton/kernel.py`, destination-passing style (output tensor last).

## Phases (see TASK.md for full gating)

| Phase | Command | Writes | Does **not** write |
|-------|---------|--------|---------------------|
| 1 Learn | `/phase1-learn` | `knowledge/` | `solution/`, `scripts/`, `bench/`, `input/` |
| 2 Implement | `/phase2-implement` | `solution/`, `scripts/bench.sh` | `knowledge/`, `context/` |
| 3 Optimize | `/phase3-optimize` | `solution/`, `ITERATIONS.md`, `solution/iterations.jsonl` | `knowledge/`, `context/`, `bench/modal_autotune.py` (unless freeze-rule acknowledged) |

`/run-all` runs all three phases sequentially in a single session — use it for a fresh end-to-end run. The per-phase commands above remain available for resuming partway through and behave identically.

A `UserPromptSubmit` hook writes the current phase slug to `.claude/state/current_phase` based on the most recent `/phaseN-*` or `/run-all` command (`/run-all` seeds `phase1`). The `/run-all` orchestrator rewrites this file at each phase transition. A `PreToolUse` hook on `Edit|Write` rejects out-of-phase writes.

## Commit-message prefixes (enforced by pre_bash hook under `solution/`)

| Prefix | When |
|--------|------|
| `[baseline]` | First correct kernel, pre-autotune (Stage 2.4). |
| `[baseline-autotuned]` | After Stage 2.5 freeze. |
| `[phase3-A] ...` | Workload classification commit. |
| `[iter N][<class_id>] ...` | Phase 3.B iteration (one technique per commit). |
| `[iter-N cleanup]` | Mandatory every 5th Phase-3 iteration, before iteration N+1. |
| `[phase3-C] ...` | Final integration commit. |
| `[phase2.5] Update modal_autotune.py for <reason>` | Autotune-script delta (must precede the sweep). |

## Artifact → producer → consumer map

| Artifact | Producer | Consumer | Schema |
|----------|----------|----------|--------|
| `knowledge/INDEX.md` | Phase 1.3 | Phase 2.3 (reference), Phase 3.B (stall rule) | — |
| `solution/subgraph_specs.json` | Stage 2.2 | Stage 2.3 (kernel impl), Phase 3.C (dispatch axes) | `context/schemas/subgraph_spec.schema.json` |
| `solution/rules_check.json` | Stage 2.4 | `tools/check_rules.py` gate | `context/schemas/rules_check.schema.json` |
| `solution/autotune_notes.md` | Stage 2.5 | Phase 3.A (representative set), Phase 3 (freeze rule) | — |
| `solution/workload_classes.json` | Phase 3.A | Phase 3.B (target class), Phase 3.C (dispatch) | `context/schemas/workload_class.schema.json` |
| `solution/iterations.jsonl` | Phase 3.B step 6 | Phase 3.B stall rule, reflexion | `context/schemas/iteration_row.schema.json` |
| `trajectory/*/bench_result.json` | Every `scripts/bench.sh` run | Phase 3.B regression guard, reflexion | `context/schemas/bench_result.schema.json` |

Always append to both `ITERATIONS.md` (human-readable) and `iterations.jsonl` (machine-queryable) on Phase-3 iterations — `tools/validate_artifacts.py` checks they stay in sync.

## Destructive ops — never

Deny rules in `.claude/settings.json` (plus a defense-in-depth check in `pre_bash.sh`) refuse:

- `git reset --hard`, `git push --force`, `git push -f`
- `rm -rf solution`, `rm -rf knowledge`, `rm -rf context`, `rm -rf bench`

If you think you need one of these, stop and tell the user.

## Pointers

- Phase gating & entry conditions — `TASK.md`
- Anti-cheat rules — `RULES.md` (machine-checked copy in `solution/rules_check.json`)
- Phase 3 performance discipline — `HINTS.md`
- Modal script reference — `bench/GUIDE.md`
- Context retrieval by bottleneck — `context/INDEX.md`

---
> Source: [Jerry2423/MoE-Kernel-Agent](https://github.com/Jerry2423/MoE-Kernel-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
