## hr-candidate-ranking-engine

> Orientation for any AI agent picking up this repo in a **new session**. Read this first,

# AGENTS.md — AI Recruiter Pipeline

Orientation for any AI agent picking up this repo in a **new session**. Read this first,
then the linked docs as needed. Keep it concise — it is loaded into context each session.

## What this is
A candidate scoring/ranking pipeline for recruiting, plus an **evaluation harness** that
gauges and regression-guards accuracy. Primary client/data: **Prime Focus Group** (UAE
HVAC/manufacturing, "HR Assistant" role). Candidate pool = a LinkedIn export (145 profiles);
JDs are free text → structured `JobRoleSchema` via an LLM.

## Current state (champion)
Weights (renormalized when a component is inactive for a JD):
`title .25 / skill .25 / qualification .05 / similarity .45 / seniority .05 / experience .05 / industry .20 / language .15 / location .05 / attrition .005 / experience_relevance .015 / education_relevance .005`
(`language` .15 is measured on the tenure/relevance/language-aware silver Judge-grade anchor: NDCG@10
.9490→.9588, with reverse metrics unchanged after language de-leak. The three structural weights are tiny
**product-motivated tie-breakers** adopted after their standalone T3 sweeps rejected larger weights: joint
NDCG@5/10 remains .9464/.9588, NDCG@20 improves .9289→.9518, and reverse MRR .5190→.5212. Original-LLM
Spearman slips .2600→.2568; fit-NDCG@10/top-k overlap hold. Tagalog-specific evidence is n=1. `location` .05
remains product-motivated. Also shipped (C5/U2):
per-role `positions[]`, a **data-completeness flag**
(rich/partial/low → screening lane, NOT in `total_score`), and the **swipe-feed card** contract (`core/swipe.py`,
`scripts/build_swipe_cards.py`) — see `docs/specs/recruiter-signals-and-swipe-feed.md`, `docs/DECISIONS.md` §C5.)
Title gate = **hybrid** (max of fuzzy & semantic), **soft** (no hard drop). Skill match = **hybrid**
(max of fuzzy char-ratio & embedding cosine; semantic floor `skill_semantic_threshold = 0.40`).
Component min-max normalization = **OFF** (tested, it hurt). **Similarity embedding** = `Qwen3-Embedding-0.6B`
(isolated to `similarity_score`; title/skill keep `all-mpnet-base-v2`), adopted **product-motivated**
(all-mpnet truncates ~44% of profiles at 384 tokens; the n=1 short gold JD can't measure the long-context
gain) — instruction-prompted, fp16, `max_seq_length=1024` (`models/mappings.py:similarity_model_config`;
set it to `None` to revert to all-mpnet). Truncation is now logged whenever inputs exceed the cap.

Honest metrics (de-leaked reverse-match + one real-JD **silver** Judge-grade anchor, n=1; Qwen champion):
- reverse: MRR **.5212**, hit@3 **.579**, hit@5 **.632**, hit@10 **.790**, seed_found 1.0
- silver anchor: NDCG@10 **.9588** / NDCG@5 **.9464** / NDCG@20 **.9518**
- two-judge agreement over the frozen 78-candidate cohort: overall Spearman **.922**; tenure **.650**;
  career relevance **.884**; preferred signals **.813**. Pipeline NDCG@10 before C5 reblend **.949** vs
  original LLM shortlist **.686**. Still circular + n=1; recruiter swipes/U2 are the un-circular check.

## How to run (uv, Python 3.11)
- Full offline eval + regenerate baseline:
  `COPILOT_SKIP_CLI_DOWNLOAD=1 uv run python scripts/run_eval.py --dataset linkedin --n-per-group 5 --out evals/results/baseline_linkedin.json`
- Tests: `COPILOT_SKIP_CLI_DOWNLOAD=1 uv run pytest tests -q --ignore=tests/test_eval_pipeline.py`
  (the ignored test hits the network / live LLM.)
- Weight tuning (offline, exact-parity): `scripts/calibrate_weights.py` — `--ablate <comp>`, `--redundancy A B`, `--joint`, or `--c5-reablate`; skill-matcher ablation: `--skill-mode {fuzzy,semantic,hybrid} --skill-semantic-threshold X`
- Real run on the HR JD + comparison to the LLM-scored pool: `scripts/run_hr_assistant.py`
- Blind LLM adjudication (live Copilot): `scripts/blind_judge_rankings.py`
- Copilot connectivity smoke test: `scripts/smoke_copilot.py` (verify auth/runtime before a live run)
- Model latency + blind-quality benchmark (JD-gen task): `scripts/bench_models.py`
- `COPILOT_SKIP_CLI_DOWNLOAD=1` keeps Copilot imports offline (tests/offline sweeps). **Unset it** for live LLM calls.

## How to test / ablate a new change
The **ablate-then-adopt loop**. Never adopt a knob/component without measuring it on the
leakage-free anchor (**gold NDCG@10**) and clearing the regression floors. Exact commands are in
*How to run* above.

1. **Implement** behind the existing seams. New component → add `calculate_<x>_score(df, jd)` in
   `core/scoring.py`, its weight in `candidate_score_weights` (`models/mappings.py`), make it
   **active-gated** inside `calculate_total_score` (it counts only when the JD supplies that signal),
   wire it into `core/pipeline.py` + `evals/runner.py` (same scorer order), and add
   `tests/test_<x>_score.py`. Tuning-only change (weight, threshold, title/skill mode) → just edit the knob
   in `models/mappings.py`.
2. **Ablate offline** with `scripts/calibrate_weights.py` (precomputes the component columns once,
   then re-combines cheaply — no LLM):
   - `--ablate <comp>_score` — 1-D weight sweep for the new component
   - `--redundancy <A>_score <B>_score` — is it complementary or redundant vs an existing one?
   - `--joint` — re-validate the core mix (sweeps title/skill/qual/similarity/seniority/experience;
     **`industry_score` is held at 0 in this mode** — ablate industry separately with `--ablate`)
   Read the **gold NDCG@10** column as the decision metric; treat reverse-match MRR as secondary/noisy.
3. **Decide.** Adopt only if gold NDCG@10 **improves or holds** AND every current floor still passes.
   A reverse-only *gain* with flat/lower gold is a leakage signal → keep the weight small or reject.
   **But first diagnose WHERE a regression comes from — the component's weight, or an eval blind spot
   — before blaming the component.** The eval can be *structurally unable* to score a new signal:
   reverse-match **invents** JD fields (location, language, …) that are NOT tied to the seed (they are
   held out of generation), so it can penalize the seed for a made-up requirement — a reverse-only
   *regression* while gold holds/improves is that artifact, not the component's fault. Instrument per
   case before judging (does the eval JD even carry this signal? is it tied to ground truth? does the
   seed match it?). If the eval can't fairly test the component, **hold that field OUT of the eval**
   (de-leak/neutralize it, like `location`/`seniority`) — do **not** reject the component or lower a
   floor to force adoption. Worked example: `location` (2026-07-12) — reverse invented a
   seed-mismatched location that dropped MRR; de-leaking it (nulled in reverse) made the sweep clean,
   then a small product-motivated weight was adopted eval-neutrally (`docs/specs/location-scoring.md`).
4. **Adopt** — set the champion weight(s) in `models/mappings.py` (active weights auto-renormalize to 1).
5. **Ratchet** — raise the affected `FLOORS` in `tests/test_eval_regression.py` to the new numbers
   (floors only go **UP**). A *methodology reset* (not a code gain) is NOT ratcheted — add a documented
   EXCEPTION note instead (see the ones already in that file).
6. **Regenerate baseline + run the suite** — `scripts/run_eval.py … --out evals/results/baseline_linkedin.json`,
   then `pytest` (expect **78 pass** + your new tests).
7. **Sanity-check (recommended)** on the real JD: `scripts/run_hr_assistant.py`; for an independent
   read, `scripts/blind_judge_rankings.py` (live LLM — **unset** `COPILOT_SKIP_CLI_DOWNLOAD`).
8. **Record it** — mark the item in `docs/BACKLOG.md`; if you made a call, add a row to `docs/DECISIONS.md`.

> Why offline ablation is ~100× faster: under soft-title the component score vectors are
> config-invariant, so the sweep runs the expensive scorers **once** per case and only re-combines
> weights — the exact `calculate_total_score` used in production.

## Key conventions
- **Adapters** (`core/adapters/*`) map any dataset → canonical `models/candidate.CandidateProfile`. Scorers read only canonical fields, never raw columns.
- **Scoring** (`core/scoring.py`): one function per component; `calculate_total_score` renormalizes the *active* component weights to sum 1.
- **Eval** (`evals/`): reverse-match (leakage-prone, secondary) + one silver gold JD (honest anchor, n=1). Metrics in `evals/metrics.py`.
- **Regression ratchet** (`tests/test_eval_regression.py`): floors only go **UP** on adoption; never lower for a code change (documented methodology-reset exceptions only).
- **JD store**: `core.jd_extraction.process_jd(jd, cache_path=...)` caches parsed JDs; `jd/parsed/*.json` are editable — tweak props and re-run with no re-extraction.
- **Caches** (git-ignored `.ai-recruiter/`): candidate embeddings are content-hash cached at `.ai-recruiter/emb_linkedin_v2.pkl`; the dir also holds Copilot session/auth state.
- **LLM providers & models**: `core.llm.get_provider(name)` — **default provider is `gemini`** (via `$LLM_PROVIDER`); pass `"copilot"` for GitHub Copilot (default model `claude-sonnet-4.5`). Pinned roles: reverse-match JD generation → `claude-opus-4.7` (`evals/cases.py:JDGEN_MODEL`, `run_eval.py --model`); blind judges → `claude-opus-4.8` + `gpt-5.5`; bench judge → `claude-opus-4.8`. `process_jd` / email generation use the provider default (no pinned model).

## Read next
- Design, module map, limitations, planned improvements → **`ARCHITECTURE.md`**
- Prioritized open work → **`docs/BACKLOG.md`**
- Decisions & rationale (don't re-litigate) → **`docs/DECISIONS.md`**
- Results → `evals/reports/pipeline_improvement_report.md`, `evals/reports/blind_comparison_report.md`
- Final shortlist deliverable → `evals/results/final_top30_combined.csv` — RRF fusion of pipeline + blind judges (top-40; generator `scripts/build_final_shortlist.py`, `--with-llm` folds in the weak original LLM). The ML pipeline alone remains the scalable product output; the fusion is the best one-off for this JD.

## Working style here
Measure before adopting: **ablate on gold NDCG@10** (the leakage-free anchor) and treat
reverse-match MRR as secondary/noisy. Adopt a knob only if it clears the regression floors.
Beware the eval's known caveats: reverse-match leakage and the **n=1 silver gold** case
(expanding it with human labels is the top backlog item). Keep changes minimal and tested.

## Agent skills

### Issue tracker

Issues and PRDs live as GitHub issues on `saishftw/ai-recruiter` (via the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Default canonical vocabulary — `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context — `CONTEXT.md` + `docs/adr/` at the repo root (created lazily). See `docs/agents/domain.md`.

---
> Source: [saishftw/hr-candidate-ranking-engine](https://github.com/saishftw/hr-candidate-ranking-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
