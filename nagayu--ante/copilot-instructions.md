## ante

> Contributions here are accepted on evidence, not on authorship.

# AGENTS.md

Contributions here are accepted on evidence, not on authorship.

We do not ask whether a patch was written by a person or by a model. That
question cannot be answered reliably, and answering it badly punishes people who
have done nothing wrong. We ask one thing instead: **is the claim your change
makes independently reproducible?**

This file is read by humans, by agents, and by the verifier itself
(`ante/policy.py` parses the block below).

## The contract

```ante
require_regression_pass: true
forbid_trivial_evidence: true
min_change_coverage: 0.34
# Mutation score is reported as a signal. Gating on it costs false positives on
# small guard-clause fixes; see the measurement in benchmarks/RESULTS.md.
mutation_kill_floor: null

evidence:
  behaviour: differential_test
  performance: benchmark_comparison
  documentation: docs_build
  typing: typecheck_delta
  refactor: mutation_delta

benchmark:
  min_repetitions: 15

first_time_contributor:
  strict_coverage: true

established_contributor:
  min_verified: 3
  strict_coverage: false
```

## What to attach

**A behaviour change** — a test that fails on the base commit and passes on
yours, whose assertions exercise the behaviour your PR describes. Put it under
`tests/`; the Action picks up test files a PR adds. Name the function or
exception you are talking about in the PR body, in backticks, so the alignment
checker can tell what the claim is about.

**A performance change** — a benchmark with at least 15 repetitions. We re-run it
on both commits and require the 95% confidence interval for the speedup to
exclude 1.0. An optimistic estimate is fine; we will correct it rather than
reject it.

**Documentation** — the examples must execute. Doctests and python fences in
changed markdown are run.

**Typing** — a measurable reduction in type-checker errors.

**A refactor** — nothing extra. A refactor claims behaviour is *unchanged*, so
the project's own suite is the right instrument, and the verifier injects faults
into your changed lines to check that the suite would still notice.

`ante bundle` produces all of this from a test run you were going to do anyway;
the marginal cost is one extra run against the base commit.

## What happens to your PR

The Action posts one of three things, and **none of them is a rejection**:

* `VERIFIED` — reproduced. A human reviews it next, with your evidence in hand.
* `NEEDS_EVIDENCE` — we could not reproduce the claim from what is attached.
  The comment says exactly what would settle it. The PR stays open.
* `NEEDS_RERUN` — your environment and ours disagree. That is drift, not doubt.

Closing a pull request is a human decision and stays one.

## Working on this repository

* `python3 -m pytest -q` — 91 tests, ~35 s.
* `python3 -m benchmarks.run` — the four-arm benchmark, ~40 s. It is
  **deterministic**: if `benchmarks/RESULTS.md` changes and you did not intend it
  to, something drifted and CI will say so.
* The corpus is entirely synthetic and must stay that way. Do not add data
  derived from a real contributor, pull request, or repository.
* Every public function's docstring names which claim it substantiates —
  maintainer load, low false positives, gaming detection, cost asymmetry, tamper
  resistance, or the privacy stance. Keep that up when you add one.

## Things this project will not accept

* Authorship or "AI-written" detection, in any form.
* Contributor scores, rankings, or a public directory of anyone's history.
* Any way to record a negative about a person. The protocol cannot express
  "this contributor's work was rejected", and it is not going to learn how.

---
> Source: [NagaYu/ante](https://github.com/NagaYu/ante) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
