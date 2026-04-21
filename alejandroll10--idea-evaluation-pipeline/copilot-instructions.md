## idea-evaluation-pipeline

> You are an AI agent running a research idea evaluation pipeline. Follow these instructions precisely.

# Idea Evaluation Pipeline — Agent Instructions

You are an AI agent running a research idea evaluation pipeline. Follow these instructions precisely.

## Overview

This pipeline evaluates a PhD student's research idea for top-3 finance journal publishability. It runs 8 steps, looping until the idea scores >= 7/10.

## Pipeline Steps

Execute steps in order. Each step has a prompt file, required inputs, and expected output.

### Step 1: EVALUATE IDEA
- **Prompt:** `prompt_ideas.txt`
- **Input:** Student's idea file + 3 closest papers
- **Output:** `[folder]/eval_[name]_idea[N].txt`
- **Web search:** Not required

### Step 2: REVIEW EVALUATION
- **Prompt:** `review_eval_prompt.txt`
- **Input:** Original idea + Step 1 output
- **Output:** `[folder]/review_[name]_idea[N].txt`
- **Decision:** If review finds critique unfair → re-run Step 1 with corrections
- **Web search:** Not required

### Step 3: PIVOT IDEA (if score < 7)
- **Prompt:** `pivot_prompt.txt`
- **Input:** Full history (idea + all previous evaluations, reviews, and pivots)
- **Output:** `[folder]/pivot_idea[N].txt` (or `pivot_idea[N]_v2.txt`, `_v3.txt` for iterations)
- **Web search:** Not required

### Step 4: EVALUATE PIVOT
- **Prompt:** `prompt_ideas.txt` (same as Step 1)
- **Input:** Pivoted idea + original 3 closest papers
- **Output:** `[folder]/eval_pivot_idea[N].txt` (or `eval_pivot_idea[N]_v2.txt`)
- **Decision:** If score dropped or stayed flat → back to Step 3 with full history
- **Web search:** Not required

### Step 5: LITERATURE REVIEW
- **Prompt:** `lit_review_prompt.txt`
- **Input:** Pivoted idea + student's 3 cited papers
- **Output:** `[folder]/lit_review_pivot_idea[N].txt`
- **Web search:** REQUIRED — every cited paper must have a verifiable URL

### Step 6: VERIFY LITERATURE REVIEW
- **Prompt:** `verify_lit_review_prompt.txt`
- **Input:** Step 5 output file (with edit access)
- **Output:**
  - Modified `[folder]/lit_review_pivot_idea[N].txt` (URLs added, fakes removed)
  - `[folder]/review_lit_review_idea[N].txt` (verification summary)
- **Web search:** REQUIRED
- **Edit access:** REQUIRED — you must edit the lit review file directly
- **Critical:** Search Google Scholar / SSRN for EVERY paper. Remove any citation that cannot be verified. This step exists because LLMs hallucinate citations.

### Step 7: FINAL VERDICT
- **Prompt:** `final_verdict_prompt.txt`
- **Input:** Full history (all previous outputs)
- **Output:** `[folder]/final_verdict_idea[N].txt`
- **Web search:** Not required

### Step 8: REVIEW FINAL VERDICT
- **Prompt:** `review_final_verdict_prompt.txt`
- **Input:** Full history + Step 7 output
- **Output:** `[folder]/review_final_verdict_idea[N].txt`
- **Decision:** If score < 7 → back to Step 3 with full history for another pivot
- **Web search:** Not required

## Loop Logic

```
if Step 2 finds critique unfair:
    re-run Step 1

if Step 4 score < 7:
    go to Step 3 (pivot again with full history)

if Step 8 score < 7:
    go to Step 3 (pivot again with full history)

if Step 8 score >= 7:
    DONE
```

Maximum 3 pivot iterations before suggesting the student try a different idea.

## Master File

Maintain a master markdown file `[folder]/[name].md` that accumulates all results. After each step, append the key findings to this file under a clear heading. The master file should read top-to-bottom as a complete narrative of the idea's evolution.

## File Naming Conventions

- Idea folder: `[name]/` (student name or idea name)
- Idea template: `idea_template.txt` (in project root — students copy this)
- Original idea: `[name]/idea.txt` (filled-in template)
- Evaluations: `eval_[name]_idea[N].txt`, `eval_pivot_idea[N].txt`
- Reviews: `review_[name]_idea[N].txt`
- Pivots: `pivot_idea[N].txt`, `pivot_idea[N]_v2.txt`, `pivot_idea[N]_v3.txt`
- Lit review: `lit_review_pivot_idea[N].txt`
- Lit review verification: `review_lit_review_idea[N].txt`
- Final verdict: `final_verdict_idea[N].txt`
- Review of verdict: `review_final_verdict_idea[N].txt`

## Parallelization

If running multiple ideas simultaneously, launch separate agents for each idea. All steps can be parallelized across ideas. Steps within a single idea must be sequential.

## Model Recommendations

| Step | Recommended | Why |
|------|-------------|-----|
| 1, 2, 3, 4, 7, 8 | Strongest available (Opus, GPT-4) | Requires deep domain knowledge |
| 5 | Model with web search | Must find real papers |
| 6 | Strongest + web search | Must verify citations accurately |

## Key Rules

1. **Never fabricate citations.** If you cannot find a paper via web search, do not include it.
2. **Always include URLs** for papers found in Steps 5 and 6.
3. **Be honest about scores.** A 6 at a good field journal is better advice than an inflated 7 at a top-3.
4. **Append to the master .md after every step.** The master file is the student's complete record.
5. **Include full history in pivot prompts.** Each pivot must know what failed before and why.
6. **Do not skip Step 6.** Citation verification is critical. LLMs hallucinate papers.

---
> Source: [alejandroll10/idea-evaluation-pipeline](https://github.com/alejandroll10/idea-evaluation-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
