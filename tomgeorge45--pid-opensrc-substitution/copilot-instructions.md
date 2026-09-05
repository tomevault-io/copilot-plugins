## pid-opensrc-substitution

> **Substitution project, not a redesign.** An existing P&ID intelligence agent already works

# PID-ML — Claude Code Context (Stage 4 Only)

## What this repo is

**Substitution project, not a redesign.** An existing P&ID intelligence agent already works
and uses cloud models (`claude-sonnet-4-6`, `claude-opus-4-8`, Google Cloud Vision) at several
stages. The goal is to replace each cloud call with the best **local** counterpart — a VLM,
LLM, or OCR model, fine-tuned per stage — then prove the local version matches the original.

**This repo is currently scoped to Stage 4 only.** Stage 4 (Symbol Detection) goes first
because it has full real ground truth and its winner becomes the **shared base VLM** every
other stage reuses. Nothing else is in scope here until Stage 4 is locked.

## Handoff discipline (standing rule, every session)

Maintain a single running file, **`HANDOFF.md`** (repo root), and keep it current — update it
at every reasonable checkpoint: after a decision is made, after finishing a meaningful chunk of
work, before context is likely to be compacted, and always before ending a session. **Overwrite
it in place** — do not create a new dated file each time. It should always answer "if this chat
ended right now, what would the next session need to know": what changed, what was decided and
why, what's open/blocked, and anything time-sensitive (e.g. files sitting in an ephemeral
scratchpad path that need rescuing before they're cleaned up).

Older one-off dated docs (`AI_Continuation_Document-*.md`, `Session_Writeup_*.md`) are historical
archive from before this rule existed — never delete them, but `HANDOFF.md` is the current
source of truth and should be read first.

## Hard rules — read before writing any code

1. **NO ML detection models.** No YOLO, RT-DETR, Relationformer, U-Net, Siamese, or any
   trained-from-scratch detector/classifier architecture. Substitutes are **local VLMs, LLMs,
   and OCR models only.**
   - The only exception: if the fine-tuned VLM can't reach the pass-bar threshold (set in
     Phase 3.6), a dedicated detector may re-enter scope for Stage 4 specifically. This is a
     documented Plan B, not a default.
2. **Do not touch offline stages.** Anything already pure Python / OpenCV / NetworkX / regex
   in the real agent is out of scope. Not our problem in this repo.
3. **One shared base VLM, not five.** Stage 4's winner is fine-tuned ONCE for domain adaptation
   (task-neutral — symbol shapes, tag formats, drawing conventions), then a **separate LoRA
   adapter** is trained on top for the detection task specifically. The domain base stays clean
   so it can be reused later at other stages via prompts alone.
4. **Real data judges. Always.** Gupta and PID2Graph are real; Kaggle is synthetic. Synthetic
   is fine-tuning volume / typing-only proxy, never the pass/fail bar for detection.
5. **Gupta is class-agnostic.** Every box is labeled "Symbol" — no type. This is why Stage 4
   uses a TWO-PART metric: detection scored on Gupta (real), typing scored on Kaggle (synthetic).
   Never average these into one number. Always report both, with Kaggle's ontology-coverage %
   next to the typing score. **⚠️ Open question (see `Agent_Pipeline_Facts.md` §3, flag 3):**
   the agent's ontology is runtime/per-tenant, not fixed — "coverage %" needs a defined
   reference ontology (e.g. an illustrative/representative set) before this can be computed
   honestly. Not yet resolved.
6. **Detection pass bar is mAP@0.5 or F1 — never recall alone.** A model that emits thousands
   of boxes scores near-perfect recall with garbage precision. Precision must be part of the bar.
7. **Test-set discipline.** The 20 Gupta test sheets are frozen in `test_ids.json` the moment
   it's built. Zero overlap with train, asserted every time, checked before every training run.
8. **No MLflow.** Use `results.csv` + `experiments/stage4/v*.md` instead — lighter weight, same
   job. See schema below.

## Read first, in this order

0. `HANDOFF.md` — the current running state of work, if it exists. Read this FIRST, before
   anything else in this list — it supersedes the older dated continuation docs as the source of
   truth for "what's going on right now."
1. `PID_Local_Substitution_Spec.md` — why this project exists, the substitution rule, the
   shared-base-VLM strategy (§5), the three VLM candidates (§5, Qwen3-VL / InternVL3 / Molmo)
2. `Stage4_Benchmarking_Checklist.md` — the actual execution plan, phase by phase, with a
   confirmation for every task. This is the authoritative to-do list.
3. `Stage4_Checklist_Status.md` — what's already done and reusable from prior work, vs. what's
   still open. Check this before starting any phase so you don't redo finished work.
4. `Agent_Pipeline_Facts.md` — **code-verified** facts about the real agent's Stage 3 tiling
   scheme, Stage 4 output schema, and entity ontology, pulled directly from
   `pnid-intelligence-agent` source. Three flags in here matter a lot: confidence and bbox are
   nested under `provenance.*` (not top-level), and the entity ontology is runtime/per-tenant,
   not a fixed enum — don't assume a flat/simple schema without checking this file first.
5. `base.md` — headline scores once runs exist.
6. `results.csv` — every run logged, one row each.

## Where we are right now

Phase 0/1 boundary. Roughly:
- ✅ Done: Drive mounted, Gupta downloaded, Kaggle downloaded+extracted+cleaned (32 classes,
  balanced, de-duplicated folder structure, train.txt/val.txt present)
- 🟡 Partial: dependencies (inspection libs only — still need torch/transformers/vllm/mlflow-
  replacement/pycocotools/supervision/kagglehub), Gupta 92-sheet count not yet asserted,
  test split identified as a concern but not frozen, two-part metric concept established but
  not built into code
- 🔴 Not started: annotation integrity check, visual box-overlay spot-check, all of Phase 2
  onward (agent-matched tiling, output-format prompt engineering, metric harness, zero-shot
  runs, fine-tuning, selection, handoff)

**Full detail in `Stage4_Checklist_Status.md` — check it before claiming any item is "TODO."**

## Immediate next actions

Phases 0 and 1 are fully complete (dependencies installed, all Phase 1 confirmations closed,
Kaggle's broken label upload found and fixed, GPU confirmed). Phase 2.1 (frozen `test_ids.json`)
is done. Agent facts pulled — see `Agent_Pipeline_Facts.md`. Next:

1. **Decide how to handle the ontology-coverage metric (2.2).** The agent's entity ontology is
   NOT fixed — it's fetched per-tenant at runtime (`Agent_Pipeline_Facts.md` §3, flag 3). There
   is no single canonical list to measure Kaggle's 32 classes' "coverage %" against, as rule 5
   and the two-part-metric section below currently assume. Needs a decision before the typing
   harness is built.
2. **Build Stage 2.3 tiling** using the now-known exact params: 1024×1024 tiles, 205px overlap,
   819px stride, title-block carve with 20px margin (reject if >30% of page), coords in
   original page-raster `[x0,y0,x1,y1]`. See `Agent_Pipeline_Facts.md` §1.
3. **Build the Stage 4 output-format harness (2.4/2.5)** using the real `DetectionRecord` shape
   — remember confidence and bbox are nested under `provenance.*`, not top-level
   (`Agent_Pipeline_Facts.md` §2, flags 1-2).

## Experiment tracking (no MLflow — use this schema)

Every run appends a row to `results.csv`:

```
run_id,date,model,stage,ft_status,detection_mAP50,detection_F1,typing_acc,rare_class_recall,parse_failure_rate,vram_gb,latency_s_per_tile,checkpoint_path,test_set_version,notes
```

- `run_id` — e.g. `qwen3vl_zeroshot`, `molmo_finetuned`
- `ft_status` — `zeroshot` | `domain_ft` | `domain_ft+detection_lora`
- `checkpoint_path` — full Drive path; must be enough for someone else to reload the exact model
- `test_set_version` — matches the frozen `test_ids.json` version, never blank

For anything needing more than a row (setup notes, gaps, hypotheses), write
`experiments/stage4/v{N}.md` using this template:

```
---
id: v1
date: YYYY-MM-DD
model: <candidate>
ft_status: <zeroshot|domain_ft|domain_ft+detection_lora>
data_version: data-v1
---
## Setup
## Results   (table: detection mAP50/F1, typing acc, rare-class recall, parse-failure rate)
## Gaps
## Hypothesis
## Next step
```

## The two-part metric (memorize this — it's the crux)

- **Part A — Detection recall/mAP/F1 on Gupta (real).** Class-agnostic: "is a symbol here?"
  Type correctness NOT judged here.
- **Part B — Typing accuracy on Kaggle (synthetic).** "Given a symbol, is its type correct?"
  Scored against Kaggle's 32 classes. Record what % of the agent's real ontology types those
  32 classes actually cover — a low coverage number means Part B tests only a fraction of the
  real type space.
- **Never average A and B into one number.** Always report both, side by side, with the
  coverage caveat attached to B.
- **Known permanent limitation:** find-and-type-correctly jointly on REAL drawings is never
  tested by this benchmark, because no real typed ground truth exists. Don't let anyone read
  "detection 0.90 / typing 0.85" as "typed detection is ~0.85 on real sheets." It isn't tested.

## Candidate models (Stage 4 base-VLM bake-off)

| Candidate | Edge | Risk |
|---|---|---|
| Qwen3-VL (7B/8B) | Best general open VLM, strong localization+OCR heritage | — |
| InternVL3 (8B) | Cleaner MIT license, genuine competitor | — |
| Molmo2-O-7B | Native pixel-pointing, strong localization | Unproven on reasoning stages (this repo doesn't touch those, but flag it in the handoff note anyway). **Switched from the original Molmo-7B-D-0924 on 2026-07-10** — the 2024-era checkpoint's custom `trust_remote_code` class hit 5 escalating incompatibilities with current `transformers` (tied-weights handling, `tie_weights` signature, generation internals). Molmo2 uses the standard `AutoModelForImageTextToText`/`generate()` pattern like the other two candidates. |

Base is selected on Stage 4 ground-truth accuracy alone. All three must go through the SAME
unified matching metric (point-in-box rule) so Molmo's points aren't scored on an easier metric
than the others' boxes.

## What not to do

- Do not build or train any object-detection architecture (YOLO/RT-DETR/etc.) unless the
  documented Plan B fallback is formally triggered per Phase 6.2.
- Do not average the detection and typing scores into one number.
- Do not use recall alone as a pass/fail bar.
- Do not skip the test-set leak assert before any training run.
- Do not score Molmo's points and the box-emitting models on different metrics.
- Do not set up MLflow — use results.csv / experiments/stage4/v*.md instead.
- Do not assume the agent's tiling/output format — pull it from the real agent code.

---
> Source: [TomGeorge45/pid-opensrc-substitution](https://github.com/TomGeorge45/pid-opensrc-substitution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
