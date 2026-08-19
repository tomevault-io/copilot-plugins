## cholesky-gpu-mode

> This is the operating manual the AI agents worked from, lightly trimmed for

# AGENTS — the standing instructions

This is the operating manual the AI agents worked from, lightly trimmed for
publication: stale mid-campaign anchors and pointers to archive-only
documents were removed, everything else is as written. Section references
like §7.29 cite the campaign's working ledger, preserved in the private
archive.

Goal: improve GPU Mode leaderboard 776, Cholesky on B200.

**Read `README.md`, then `RULES.md`, then `experiments.md` before doing
anything.** `RULES.md` is binding and is not repeated here.

## Context budget — read this before opening any file

The documentation is deliberately cheap; the code is not. The maps exist so you
never have to open a candidate whole.

| Tier | What | Cost | When |
|---|---|---:|---|
| Always | `AGENTS.md` | ~1.5k | auto-loaded |
| Task start | `README.md` + `RULES.md` + `experiments.md` | ~8.8k | every task |
| Before touching code | `ARCHITECTURE.md` | ~3.5k | any change to a candidate |
| Before proposing | `logs/campaigns.md` — all 35 campaigns | ~7.6k | **anything you think is new** |
| Reference | `logs/profiles.md` · `logs/archive/superseded.md` | 9.1k · 100k | hardware data · provenance |
| **Never bulk-read** | `candidates/*.py` | **~230k for `108`** | grep / ranged-read only |

Everything you routinely need is **~24k tokens including the full experimental
history**. The champion alone is ~230k.

**The rule that makes this work: never `Read` a candidate without `offset` and
`limit`.** The champion (`108`) is **19,482 lines and 55% embedded
CUDA**.
`ARCHITECTURE.md` §2 gives the line span of every block — pick the range, read
that. To find a gate or a symbol, `grep -n` first.

Two facts that cost time if you learn them the hard way, both in
`ARCHITECTURE.md`:

- `custom_kernel` is defined **twice**; the second definition shadows the first.
  The one you find first is not the entry point. (Line numbers in
  `ARCHITECTURE.md` are `071`'s — the champion is 18,565 lines. **`grep -n`.**)
- `_CLU_CUDA` is **not** the dead cluster family — it is the base translation
  unit every engine appends onto.

---

## The two objectives

You are optimising **ranked geomean SUBJECT TO an 8/8 validation pass**. Both
must hold. A file that is faster but validates <8/8 is not an improvement, it is
a regression — `submission_063` is 6.8% faster than the champion and unshippable.

| | Ranked | Validation |
|---|---|---|
| Harness | `harness/modal_eval.py` | `harness/modal_validate.py` |
| Shapes | 15 benchmark entries | 8, a subset |
| Metric | geometric mean of runtime | pass/fail per shape |

Full contract detail: `RULES.md` §3.

## Problem / metric

Input: `batch × n × n` CUDA `float32` SPD matrices.
Output: lower-triangular `float32` factor `L` with positive diagonal, `A = L Lᵀ`.

The checker validates shape, dtype, device, finiteness, lower-triangular
structure, positive diagonal, and reconstruction residual against the original
FP32 input. Correctness is property-based, not elementwise against one library.

Ranking is the **geometric mean** over 15 benchmark entries. Because the geomean
is scale-free, a 2× improvement on any one shape gives the same benefit
regardless of absolute microseconds — so **rank levers by how many cases one
mechanism reaches, not by microseconds**, and never let one route regress.

## Benchmark grid

Case index mapping for `modal_eval.py --cases`:

| Case | Shape | | Case | Shape |
|---:|---|---|---:|---|
| 0 | `4096 × 32` | | 8 | `2 × 2048` |
| 1 | `1024 × 64` | | 9 | `8 × 2048` |
| 2 | `256 × 128` | | 10 | `1 × 4096` |
| 3 | `64 × 256` | | 11 | `2 × 4096` |
| 4 | `16 × 512` | | 12 | `1 × 8192` |
| 5 | `640 × 512` | | 13 | `1 × 16384` |
| 6 | `4 × 1024` | | 14 | `1 × 32768` |
| 7 | `60 × 1024` | | | |

Validation shapes are cases 0, 1, 2, 3, 4, 6, 8, 10 — but with a completely
different data distribution. See `RULES.md` §3.

## Experiment discipline

1. Read `experiments.md` §5 (what worked) and §6 (what did not) **first**, then
   `logs/campaigns.md` for the measurement behind any verdict. §6 is long because
   each row cost real time; re-running any of it is pure waste.
2. Pick one exact route / shape / hypothesis.
3. Clone the current champion into a new candidate file.
4. Make **one** focused change, as an **additive default-OFF gate**, and prove
   gate-OFF translation-unit byte-identity.
5. Validate correctness before benchmarking.
6. Benchmark subset cases first, full grid only if promising.
7. Run `modal_validate.py` — 8/8 or it does not ship.
8. Record the result in `experiments.md` **including negatives**, with candidate,
   hypothesis, result, verdict, next action.
9. Never submit officially. Tell the user which file is ready.

## Before proposing anything

Check it against `experiments.md` §6 and `logs/campaigns.md` §IV — "closed with
data". Twenty-plus mechanisms are already closed with measurements. Do not
re-propose one without new information, and say what the new information is.

Check it against §7 — "measurement landmines". In particular:

- **Run `-Xptxas -v` on every candidate BEFORE quoting any A/B number** (§7.29).
  If **any persistent flow kernel's** register count moves — ⚠ **including the
  one your change deliberately modifies** (§7.33) — the A/B is unconfirmed.
  `N2F` moved `_rl_flow_kernel` 110→112 and missed by 3.1 pp; **`R128SP` moved
  `_r128_flow_kernel` 128→109 — the "good" direction — and missed by ~4 pp**;
  `N2LD=5` and `RLGRID_SHAPES` moved none and hit to 0.07 and 0.04 pp. **4 for
  4.** ⚠ **Also §7.33: never spend a submission on a gain that lives only in
  c10/c11** — three r128-only changes, three official failures. **Register-identical is necessary but not
  sufficient** (§7.31): inlined device functions can move the SASS at an
  unchanged register count, so diff the instruction mix too.
- **Never quote a sub-1% figure from a multi-`modal run` A/B** (§7.23).
  Separate containers confound arms; use `harness/modal_ab.py`, which
  interleaves both arms in one container and reports each arm's self-spread.
- **After any speedup, re-sweep the constants that encode a balance** (§7.30).
  `RLGRID_SHAPES` paid −0.49% for a one-line host-side diff because our own
  wins had invalidated it — the third instance of that same coupling. Per-shape,
  never blanket: uncapping the other shape in the same A/B cost +4.1%.
- ⚠ **§7.22's "only spend a submission on a c0–c9 gain" is SUPERSEDED** by
  §7.23. It was an instrument artifact, not a property of the large cases.
  `submission_097` won on c12/c13/c14 alone. **All 15 cases are in play.**
- **Transfer is mechanism-dependent, and there are now THREE classes:**
  kernel/occupancy/precision **1.0–1.94×**; **sync / launch-pipeline ~0.5×**
  (§7.15 — `NFDEV` measured −9.20% on Modal and delivered −4.50% officially);
  RL-engine *routing* **~0**. Classify your change before quoting any Modal
  number. Every estimator overshot `NFDEV`, including the pessimistic one.
- **A green 17/17 on the public test grid means little** — it has no case with
  batch ≥ 64, so it cannot execute anything gated to 640×512 or n ≥ 4096.

## Naming

Files handed to the user for submission: `submission_NNN.py`, continuing from
the champion. Neutral names only — no route names, shapes, or strategy in the
filename. Descriptive notes go in `experiments.md`.

Working probes may use a mnemonic name, but **do not keep both a probe file and
its submission file** unless they differ by more than a gate default. The old
tree accumulated 650 KB near-duplicate pairs differing by a single character;
that is what this consolidation removed. **A later campaign re-accumulated
19 such files (14 MB) before they were deleted again** — an A/B probe pair is
scratch, not a candidate. Build them under the scratchpad directory, or delete
them the moment their number is in `experiments.md`.

## Modal

**Always run from the repository root.** The images build with
`add_local_dir("problem", ...)`, which resolves relative to the working
directory, not to the script.

Modal may not be installed in the active local Python environment. If a command
fails with `ModuleNotFoundError: modal`, use the project environment that has
Modal configured, or ask the user to set it up.

```bash
modal run harness/modal_eval.py     --mode test      --candidate candidates/<file>.py
modal run harness/modal_eval.py     --mode faithful  --candidate candidates/<file>.py
modal run harness/modal_eval.py     --mode benchmark --candidate candidates/<file>.py --cases "0,1,2"
modal run harness/modal_validate.py                  --candidate candidates/<file>.py
```

Use `--cases` subsets while iterating. Run the full 15-case benchmark only after
correctness passes. **Both test modes must pass with no `--env`.**

For effects under ~1%, `modal_eval.py` cannot resolve the difference —
use `modal_ab.py` (single container, interleaved arms, self-spread reported).
The campaign's other measurement harnesses (compile audits, seed sweeps,
import timing, phase profilers) live in the private archive.

---
> Source: [ravi03071991/cholesky-gpu-mode](https://github.com/ravi03071991/cholesky-gpu-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
