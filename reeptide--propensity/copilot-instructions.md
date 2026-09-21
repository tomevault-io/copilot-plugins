## propensity

> This project has a design (`DESIGN.md`) and is now **v1-released**: Phases

# CLAUDE.md — Context for AI Assistants

This project has a design (`DESIGN.md`) and is now **v1-released**: Phases
1-4 (§10) are complete with real results, `README.md` is published, and the
repo is git-tagged `v1`. This file records **why** the design looks the way
it does, so a fresh session doesn't re-litigate settled decisions or
re-propose rejected ones.

Read `DESIGN.md` first, then `README.md` for the actual results. This file is
the reasoning behind the design.

---

## Current status (2026-09-09, v1 tagged + audited, Phase 5 appendix built)

**Environment:** venv named `expedia` (not `.venv`) at the project root —
`source expedia/bin/activate`. Package installed editable (`pip install -e .`)
as `ranking`, importable from `src/ranking/`. Both datasets downloaded and
extracted under `data/expedia/` and `data/trivago/`; `data/processed/` fully
materialised. `docker` + `docker compose` present and verified working.

**Done — Phases 1-5.** Numbers below are post-audit (an independent review
found and this repo fixed a broken calibration model, a corrupted propensity
curve, a SNIPS rank/position bug, and a `.gitignore` that excluded the entire
`src/ranking/models/` package from git — see the audit commit for the list).
- Phase 1: arms 0/1/1b/2/3 on both datasets — `results/phase1_expedia.md`,
  `results/phase1_trivago.md`.
- Phase 2 (Deliverable A): full-scale propensity/IPW/SNIPS/calibration run —
  `results/phase2_expedia.md`. Headline: IPW beats naive LambdaMART by
  **+0.0028 NDCG@38** [+0.0002, +0.0053] on the sealed randomised holdout —
  real but modest: MRR agrees (+0.0038 [+0.0004, +0.0078]), NDCG@10 does not
  clear significance, and SNIPS (independent estimator) agrees on direction.
- Phase 3 (Deliverable B): session ranking arms, retrieval coverage over the
  full 927k catalogue, MMR re-ranking — `results/phase3_trivago.md`. Session
  features are the biggest lever (MRR 0.47 → 0.61); the fully-stacked ranker
  (session + kNN + SASRec) is slightly *below* session-features-alone — an
  honest null result, kept rather than hidden.
- Phase 4: promotion gate simulation (`results/promotion_gate.md`) and load
  test against the live serving stack (`results/loadtest.md`).
- `README.md`: results tables, architecture diagram, `docker compose up`
  reproduce (verified end-to-end), latency/fallback numbers, shipping
  recommendation, limitations.
- Phase 5 (appendix, post-v1 by design): **arm 6 LightGCN**
  (`results/lightgcn.md`, `src/ranking/models/lightgcn.py`) and the **RQ-VAE
  semantic-ID tokenizer** (`results/semantic_ids.md`,
  `notebooks/semantic_ids.ipynb`, `src/ranking/rqvae.py`) — both labelled as
  learning artifacts, neither producing a shipping recommendation. Two
  findings there were NOT predicted and are the reason the arm earned its
  place: on cold-positive clickouts LightGCN actively promotes the warm items
  it knows above the correct one (worse than displayed order 19% of the time,
  i.e. "confident wrong opinion", not "no opinion"); and training it 5x longer
  improved its own BPR objective +47% while making the ranking metric worse,
  which is a train/eval objective mismatch (uniform catalogue negatives vs
  discriminating within a ~25-item impression list), not a model-class ceiling.
- 63/63 tests pass (`pytest tests -q`), no warnings.

**Two real bugs found and fixed while wiring Phase 2-4 together, worth
knowing about if touching serving code:**
1. `serving/pipeline.py::build_frame()` never computed `knn_score`/
   `sasrec_score`, so the originally-configured primary model
   (`lambdamart_full`, which needs those features) silently failed every
   request and fell back to item-kNN even when "healthy". Fixed by making
   `lambdamart_session` the default `PRIMARY_MODEL` (it's also the
   better-measured Phase 3 arm) rather than adding live kNN/SASRec scoring
   to the hot path. `lambdamart_full`'s online-serving gap is documented in
   README as a known limitation, not hidden.
2. `docker compose up` had never actually been exercised: `python:3.11-slim`
   is missing `libgomp1` (LightGBM's compiled booster needs it), and
   `README.md` didn't exist yet even though the Dockerfile `COPY`s it. Both
   fixed; a full build+up+`/rank` round-trip is now verified working.

**Remaining — optional, post-v1 per DESIGN.md's release gate:**
- Phase 5 (appendix): LightGCN warm-subset confirmation, RQ-VAE notebook.

## What this is

A recommender-systems portfolio project for **applied DS / MLE roles at travel and
e-commerce companies** (Expedia, Booking, Flipkart, Amazon).

Two datasets, two self-contained deliverables:
- **Expedia ICDM 2013** → unbiased learning-to-rank (position-bias correction)
- **Trivago RecSys 2019** → session-based ranking + a production serving layer

## Who it's for

Solo developer, targeting industry hiring — **not** academic publication.
Hardware: single 8GB RTX 5050. Time is not a hard constraint; **motivation over a
long project is the real risk.** Prior project (`../pho2rec`) stalled at step 5 of
14, which is why the design has a v1 release gate.

Demonstrated existing strengths, all of which port to this project: FastAPI, async
SQLAlchemy, PostgreSQL, Redis, React/TypeScript, OpenCLIP embedding pipeline, a
pluggable ranker registry. **The serving layer is a strength, not a stretch** —
lean on it.

---

## Rejected approaches — do not re-propose

Each of these was considered seriously and rejected for a specific reason.

| Rejected | Why |
|---|---|
| **pho2rec** (personal photo album recommender) | Structurally unfixable: no training data and no way to get any. ~15–25 sessions from 1–4 users. Collaborative filtering out of scope by design, so popularity is degenerate and there's no user×item matrix. Now shelved; finish separately as a plain photo-browser app with **no ML claims**. |
| **H&M Kaggle fashion recommender** | 1st-place solution explicitly reported image/text features did not improve ranking. Winning MAP@12 was 0.0379; 1st→45th gap is 0.008, so effects live in the 3rd–4th decimal. Also one of the most-done portfolio datasets on GitHub. |
| **"Do visual features help cold-start?"** | Settled since Schein et al. (SIGIR 2002) and VBPR (He & McAuley, AAAI 2016). Not a hypothesis — it's the design rationale of a model class. |
| **Visual encoder comparison** (CLIP/SigLIP2/DINOv2/DINOv3) | Trains nothing, engineers no features, makes no modelling decisions. Correctly identified by the developer as "hardly a data science project." |
| **Two-tower + logQ / negative-sampling ablation** | All four planned ablation arms are settled: logQ by Yi et al. (RecSys 2019); loss choice and negatives-count by Klenitskiy & Vasilev and gSASRec (both RecSys 2023); the sampling × cold-start interaction by Prakash et al. (RecSys 2024 workshop, arXiv 2410.17276). |
| **Semantic IDs / TIGER for cold start** | Empirically refuted — arXiv 2607.21101 (July 2026) reports unseen-item Recall@20 of 0.00000 under proper temporal splits. |
| **Reproducibility study** ("how much of published cold-start gain survives correct evaluation?") | Genuinely strong as research, but the developer explicitly wants a practitioner project, not a paper. Rejected on that basis, not on quality. |
| **OTTO dataset** | No impression logs — positives only. Position bias unstudiable. |
| **Sponsored-slot injection** | No auction, no bids, no revenue data. Any number would be fabricated. |
| **Full generative recommender / LLM reranker** | Weeks of work on 8GB, nothing visible in a demo, thin item metadata on Trivago. |
| **Kubernetes** | Overkill and unverifiable at this scale. Docker Compose only. |

**Standing lesson from this project's history:** three separate "obviously open"
research framings turned out to have 2024–2026 papers on them. This subfield is
far more thoroughly worked than it looks. If a research angle seems novel,
assume it isn't until checked. This is also why the project is now framed as a
practitioner build rather than a contribution.

---

## Decisions that took work — don't undo them

**Why two datasets.** A debiasing intervention cannot be evaluated on a biased test
set — a biased metric rewards reproducing the incumbent ranker, so IPW will often
*lose* with no way to tell method failure from metric failure. Expedia's
randomised-ordering subset is the only unbiased holdout available. Hence: Expedia
owns the entire position-bias arc; Trivago owns sessions + serving and makes no
IPW claim.

**Why the displayed-impression-order baseline (arm 1b) matters most.** It's the
first question a ranking interviewer asks. On Trivago it's a very strong baseline.
Without it every downstream number is suspect.

**Why retrieval recall is instrumented separately.** Retrieval sets the ceiling.
Not measuring it is how people waste weeks tuning a ranker over a candidate set
that never contained the answer.

**Why the v1 gate exists.** The dominant risk is a half-built repo — a design doc
that outruns the code reads as "planner who doesn't ship." Tag `v1` at the end of
Phase 2; no appendix work before that.

**Why LightGCN and the RQ-VAE notebook survived cuts.** The developer's criterion
is "cheap + I learn something," not pure signal-per-hour. Both are scoped and
sit in Phase 5, behind the v1 gate. LightGCN is framed as a *confirmation with a
number*, not a discovery, because its transductive failure on cold items is
predictable in advance.

---

## OPEN QUESTIONS — resolve before Phase 1

**1. The synthetic IPW validation currently FAILS, and it is unresolved.**

`scripts/phase0_derisk.py` check 5 trains LightGBM lambdarank on position-biased
synthetic clicks, with and without IPW weights, and scores both against known
true relevance. On the first run: naive NDCG@10 = 0.8958, IPW = 0.8912, delta =
**−0.0046**. IPW did not help.

Two candidate explanations, not yet distinguished:
- **(a) LightGBM weight semantics are wrong** — `weight` may not interact with
  lambdarank query groups the way intended. This is the failure the check exists
  to catch, and it would be interview-ending if it reached the real headline.
- **(b) The synthetic generator is too benign.** With `eta=1.0` over 20 docs,
  IPW weights span 1–20, which is a large variance penalty; and because the
  logging policy ranks by noisy true relevance, position bias attenuates the
  signal without inverting it, so a naive model can still recover the
  feature→relevance mapping. IPW may simply have little to recover.

A diagnostic was written to distinguish these (sweep `eta` ∈ {0.5, 1.0, 2.0} ×
logging policy ∈ {noisy-relevance, feature-subset-confounded}, plus a sanity arm
using random extreme weights to confirm weights change the model at all) but the
scratchpad was cleared before it ran. **Rebuild and run it.** Expected shape: if
random extreme weights barely move NDCG, weights are being ignored → (a). If a
confounded logging policy at higher `eta` shows IPW clearly winning, the original
generator was benign → (b), and the check's thresholds need retuning.

**Update 2026-08-02 — (b) and a third candidate, (c) weight variance, are now
ruled out.** Ran the full sweep (`--ipw-eta 2.0 --ipw-confounded`, `--ipw-eta 0.5
--ipw-confounded`) plus a new weight-clipping arm added to check 5
(`phase0_derisk.py`). Findings:
- Making the logging policy harsher/more confounded made IPW's deficit *worse*
  (−0.0046 → −0.0200), not better. Rules out (b) — a too-benign generator would
  show IPW improving as bias severity increases; it did the opposite.
- Clipping propensity weights at p95 changed nothing (delta moved by <0.001 in
  every scenario). Rules out (c) — if a handful of huge-weight outliers were
  destabilising training, clipping would have recovered most of IPW's advantage.
- The random-extreme-weights sanity arm (weights spanning 1e-3 to 1e3, a
  million-fold range) only ever moved NDCG@10 by ~0.006–0.011 across all
  scenarios. That is suspiciously small for a million-fold weight spread.

**Leading hypothesis is now (a), specifically:** LightGBM's `lambdarank`
objective does not strongly honour per-instance `weight` in the pairwise ranking
loss itself — `weight` is documented to matter more for tree-growing (split
selection) than for the ranking gradient. This is a known property of the
implementation, not a bug in this project's code.

**Update 2026-08-02 (cont'd) — the weighted-label fix was tried and does NOT
resolve this; revised diagnosis.** Implemented and tested the standard
alternative: bake the IPW correction into the training TARGET instead of a
`weight=` argument, i.e. train a plain regressor (`objective: regression`, L2)
on `pseudo_label = click / propensity` (unbiased for true relevance in
expectation), with and without flooring the propensity denominator at p5 to cap
pseudo-label magnitude. Result, same three-scenario sweep:

| Scenario | naive | `weight=` IPW | weighted-label | weighted-label + floor |
|---|---|---|---|---|
| mild bias (eta=1.0) | 0.8958 | −0.0046 | **−0.0606** | −0.0575 |
| harsh + confounded (eta=2.0) | 0.8160 | −0.0200 | **−0.1432** | −0.1435 |
| weak + confounded (eta=0.5) | 0.8695 | +0.0003 | **+0.0100** | +0.0094 |

The weighted-label fix is *worse* than the original `weight=` approach in two
of three scenarios, and flooring the propensity made no meaningful difference.
This rules out "`weight=` is specifically broken" as the primary story — a
mechanism that bypasses `weight=` entirely fails even harder.

**Revised diagnosis: this is a high-variance IPS estimator problem, not an
implementation bug in either mechanism.** A click at position 20 under
`eta=2.0` produces a pseudo-label/weight of ~400 vs. ~1 elsewhere — an
extreme, sparse spike that dominates a squared-error or weighted loss when
there are only ~4,000 synthetic queries to average over. This is a
well-documented failure mode of vanilla inverse-propensity-weighting in the
off-policy learning literature (high variance with small propensities / small
sample sizes), not specific to LightGBM.

**Implication for Deliverable A:** vanilla IPW (whether via `weight=` or via
weighted labels) is not safe to use as-is. The next things to try, in order of
standard practice: (1) self-normalized IPW / SNIPS-style normalization within
each query group (already planned for *evaluation* in DESIGN.md §6 — extend the
same idea to *training*); (2) a doubly-robust estimator (combine IPW with a
direct relevance-regression model to reduce variance); (3) re-run this same
synthetic check at a much larger `n_queries` to see whether the variance
problem is partly a small-sample artifact of the synthetic generator itself
before concluding it will bite equally hard on the real ~120k-query randomised
Expedia subset, which is 30x larger.

This is now a load-bearing open question for Phase 2, not a solved bug —
resolve one of the three options above and re-validate against this same
synthetic harness before trusting any real IPW result.

**Update 2026-08-02 (RESOLVED) — it was a small-sample artifact of the
synthetic test, not a real IPW/LightGBM problem.** Tried option (3) above:
reran the plain `weight=` IPW check (no clipping, no SNIPS, no label tricks —
the original, simplest formulation) at `n_queries` = 4,000 / 40,000 / 120,000,
holding `eta=2.0, confounded=True` fixed:

| n_queries | naive NDCG@10 | IPW NDCG@10 | delta |
|---|---|---|---|
| 4,000 (original check 5 default) | 0.8160 | 0.7960 | −0.0200 |
| 40,000 | 0.8193 | 0.8235 | +0.0042 |
| 120,000 (≈ real Expedia randomised-subset size) | 0.7282 | 0.7486 | **+0.0204** |

Plain IPW clearly wins once query count matches realistic scale. The self-
normalization, clipping, and weighted-label experiments above were solving a
problem (variance at n=4,000) that mostly evaporates at n=120,000 — the actual
scale of the real randomised subset (check 1). **Conclusion: `Dataset(...,
weight=1/exam_prob)` under `lambdarank` is fine to use as originally written.**
The one actionable follow-up: `check_5_synthetic_ipw`'s default `n_queries=4000`
in `phase0_derisk.py` is miscalibrated for this project's actual scale and
should be raised (e.g. to 100k+) so the check's own default run doesn't produce
a false-alarm INCONCLUSIVE/FAIL in future sessions.

**2. Verify the datasets before committing.** Randomised-subset fraction, whether
Kaggle late submission is open (Expedia), and regression-EM identifiability. All
covered by the Phase 0 script.

**3. `random_bool` split protocol across Phase 1/2 — RESOLVED 2026-08-02.**
Got a second opinion (Opus subagent) on whether Phase 1's baseline arms (0–3)
should train on all of `train.csv` or only `random_bool=0`. Decision: **only
`random_bool=0`, for the whole of Phase 1** — full reasoning and the resulting
two-way split of the `random_bool=1` holdout (calibration slice vs. sealed
headline slice, to avoid tuning the propensity estimate on the same rows used
to grade it) is written into `DESIGN.md` §5 ("`random_bool` split protocol").
This is a load-bearing decision for Phase 1 ingestion code — implement the
split this way from the start rather than retrofitting it.

**4. Trivago temporal split boundaries — CORRECTED 2026-08-02.** DESIGN.md
originally planned days 1-6 train / 7 val / 8 test. Verified against the real
`train.csv`: it only spans days 1-6 (confirmed via timestamp range, both
independently and aligned against `test.csv`'s timestamp range, which picks up
at day 6 and runs to day 8). Days 7-8 exist only in `test.csv`, and there,
each session's FINAL clickout has `reference` set to null — that's the Kaggle
prediction target, not usable ground truth. There is no way to get fully
labelled days 7-8 data in this offline setup (consistent with the earlier
finding that both official Trivago hosting domains are dead and no
`test_ground_truth.csv` is available anywhere).

**Fix:** split is now days 1-4 train / 5 val / 6 test, entirely within
`train.csv`. Verified this doesn't cost real evaluation power — per-day
session counts are uniform (137k-162k, no outlier day), so val/test set sizes
are essentially unchanged from what days 7/8 would have given; only the
training window shrinks from a hoped-for 6 days to 4 (587k sessions), which is
still ample at this dataset's scale. Implemented in
`src/ranking/splits.py::temporal_split_trivago` (defaults changed from
`train_days=(1..6), val_day=7, test_day=8` to `train_days=(1..4), val_day=5,
test_day=6`) and verified against real data via
`scripts/validate_phase1_splits.py`.

---

## Working preferences

- The developer pushes back hard on weak reasoning and is usually right to. Two
  significant course corrections in this project came from their objections, not
  from review.
- Be blunt about expected null results and state them in advance.
- Prefer honest limitations over impressive-sounding claims — the design
  deliberately reports what it cannot establish.
- Independent review has been valuable here; several structural errors were caught
  that way. Use it when a decision is load-bearing.

## Environment

- Linux. `pyproject.toml` requires >=3.11 (also what the Dockerfile/CI pin for the
  served image); the actual `expedia/` venv that produced every result in `results/`
  is **3.13.13** (`expedia/pyvenv.cfg`), not 3.11 — this was a stale claim in an
  earlier version of this doc, corrected 2026-09-09.
- `scripts/phase0_derisk.py` needs `polars`, `numpy`, `lightgbm` — install into a
  venv or a fresh conda env, not the `pho2rec` env.
- Datasets are Kaggle-gated and not yet downloaded. Nothing has been run against
  real data.

---
> Source: [Reeptide/propensity](https://github.com/Reeptide/propensity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
