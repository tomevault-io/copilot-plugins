## socialai

> - USC CARC SLURM cluster

# Allegro Project — Claude Instructions

## Environment
- USC CARC SLURM cluster
- Package manager: uv (NOT conda, NOT pip directly)
- Python: managed by uv, pinned in pyproject.toml (>=3.11)
- Virtual env: `.venv/` at project root (managed by uv)

## Cluster Info
- **Partitions:** `nlp_hiprio` (high priority, no preemption) and `nlp` (preemptable)
- **Account:** `robinjia_875`
- **GPUs:** ~60 A6000s, 4 A100s
- **Default resources per job:** 8 CPUs, 1 GPU, 40G RAM
- **Max walltime:** typically 2 days default in templates

## How to Run Things

## Behavioral Categories (9 total)

Each measure is a folder under `src/filter/measure/<name>/` containing
`filter1.json` … `filter4.json`. Measures are grouped into three principles
by the leading digit of the folder name (used for near-miss pooling in
synthetic generation):

| Principle | Theme | Measures |
|---|---|---|
| **1x** | Identity / human-like presentation | `1B_intentional_human_speech`, `1B_human_pronoun`, `1C_identity_transparency` |
| **2x** | Fabrication, emotional expression, sycophancy, relationship encouragement | `2A_fabricated_personal_information`, `2B_emotion_expression`, `2C_deference`, `2C_flattery_tone`, `2D_human_relationship_encouragement` |
| **3x** | Engagement | `3A_engagement_hooks` |

Authoritative in-the-wild seedset files:
- `data/seedset_raw.jsonl` — 322 rows, 368 original tags; copied locally
  from the manually consolidated jessetho mirror file
  `data/new_seedset.jsonl`.
- `data/seedset_data.jsonl` — same 322 rows after the Stage 6 union
  relabel pass, 960 tags. This is the in-the-wild portion of
  `data/final_dataset.jsonl`.

The active Stage 7 input is `data/final_dataset.jsonl`: 969 rows
(322 in-the-wild + 647 manually accepted synthetic), 3,147 total tags,
schema `{user_input, measure, synthetic, language}`.

### Install dependencies
```
uv sync
```

### Run a filter stage directly (on a compute node with GPU)
```bash
uv run python src/filter/run.py \
    --measure <measure> \
    --stage <stage> \
    --input_path <input.jsonl> \
    --output_path <output.jsonl> \
    [additional stage-specific args]
```

`--measure` is the folder name under `src/filter/measure/` (e.g., `1B_intentional_human_speech`, `2C_flattery_tone`).
`--stage` is the Python file name within that folder (e.g., `coarse_filter`, `low_quality_filter`, `high_quality_filter`, `final_filter`).

The optional `--experiment_dir` flag snapshots `src/filter/` into `<experiment_dir>/src/` before running (for version control).

### Submit a SLURM job
Run filter stages via pipeline.sh:
```bash
bash pipeline.sh --measure <measure> --stage <stage> [--shards 6] [--input <path>]
```
This submits the SLURM job(s) for one stage and creates `experiments/<NN>_<stage>/` for logs and snapshots.

Generic SLURM templates for simple single-script jobs are in `slurm/`:
```bash
sbatch slurm/run_gpu.sbatch <script.py>        # single job, high priority
sbatch slurm/run_preempt.sbatch <script.py>    # single job, preemptable
```

Array jobs set `SLURM_ARRAY_TASK_ID` env var; experiments can read it via `os.environ.get("SLURM_ARRAY_TASK_ID")` (returns `None` for non-array jobs).

### Module loads needed (already in sbatch templates)
```
module purge
module load gcc/13.3.0
module load cuda/12.6.3   # must be 12.6.3 — .venv was built with torch+cu126
```

### Start the vLLM server (on a compute node with GPU)
```
uv run vllm serve Qwen/Qwen3-VL-8B-Instruct --port 8001 --max-model-len 16384 --gpu-memory-utilization 0.85
```
Stages 1 and 2 (`coarse_filter`, `low_quality_filter`) use a local vLLM server via the `VLLM_PORT` env var (default `8001`).

### Run tests
```
uv run python -m pytest --extra dev
```

## Full Pipeline

The pipeline has three phases: **filtering** (Stages 0-4, run per measure), **preprocessing & synthetic generation** (Stages 5-6, run once across all measures), and **evaluation & analysis** (Stage 7 substages 7.1-7.3: generate model responses, judge, and analyze).

Always submit from the project root.

### Phase 1: Filtering (per measure, via pipeline.sh)

Run Stages 1-4 for each measure (e.g., `1B_intentional_human_speech`, `2C_flattery_tone`, etc.).

**Stage 0 — Download WildChat (once, CPU job):**
```bash
sbatch slurm/run_download.sbatch
```
Downloads `allenai/WildChat-4.8M` (the **public, non-gated** release — contains ~3.2M conversations, *not* the full 4.8M which lives in the gated `allenai/WildChat-4.8M-Full` repo), filters English, deduplicates by `conversation_hash` → `data/wildchat_raw.jsonl` (1,442,077 rows).

**Stage 1 — Coarse Filter (vLLM, Qwen3-VL-8B):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage coarse_filter
```
Keyword/pattern-based filtering using local vLLM. Uses `filter1.json` prompts.
Creates `experiments/<NN>_coarse_filter/` with output in its `results/` subfolder.

**Stage 2 — Low-Quality Filter (vLLM, Qwen3-VL-8B):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage low_quality_filter \
    --input experiments/<NN>_coarse_filter/results/1B_intentional_human_speech_coarse.jsonl
```
Domain-specific judge using local vLLM. Uses `filter2.json` prompts.

**Stage 3 — High-Quality Filter (OpenRouter API, GPT-4o-mini):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage high_quality_filter \
    --input experiments/<NN>_low_quality_filter/results/1B_intentional_human_speech_scores.jsonl \
    --key <path/to/openrouter_api_key>
```
Dual-check (chitchat + category) with GPT-4o-mini. Uses `filter3.json` config.
Output contains `chitchat_keep` and `category_keep` fields.

**Stage 4 — Final Filter (OpenRouter API, Claude Opus 4.6):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage final_filter \
    --input experiments/<NN>_high_quality_filter/results/1B_intentional_human_speech_high_quality.jsonl \
    --key <path/to/openrouter_api_key>
```
Re-evaluates only the intersection rows (both `chitchat_keep=true` AND `category_keep=true` from Stage 3) with Claude Opus 4.6. Uses `filter4.json` config.

### Phase 2: Preprocessing & Synthetic Generation (once, across all measures)

After running Stages 1-4 for all measures, collect, dedupe, and augment with synthetic data.

**Stage 5 — Data Preprocessing + Seedset Consolidation (CPU + OpenRouter API):**
```bash
uv run python src/data_preprocessing/data_preprocessing_v2.py \
    --output data/final_v2.jsonl \
    --key <path/to/openrouter_api_key>
```
`data_preprocessing_v2.py` records the current v2 preprocessing procedure:
it reads a curated cross-repo source list, keeps Stage 4 both-KEEP rows,
deduplicates by `user_input`, and batch-classifies single vs multi-turn
messages with Opus 4.6. The direct v2 output was then manually
consolidated in the jessetho mirror into `data/new_seedset.jsonl`, copied
locally as `data/seedset_raw.jsonl`.

Current authoritative Stage 5 artifacts:
- `data/seedset_raw.jsonl` — 322 in-the-wild rows, 368 original tags.
- `data/seedset_data.jsonl` — same 322 rows after union relabel, 960 tags.

Legacy `data_preprocessing.py`, `data/final.jsonl`,
`single_turn_final*.jsonl`, and `multi_turn_final.jsonl` describe an older
single/multi-turn pipeline and are not the active Stage 7 input.

**Stage 6 — Synthetic Generation (OpenRouter API, Opus 4.6 rewriter):**
Rewrite "near-miss" user inputs (rows that fell out at Stages 3-4) into
on-target candidates, anchored to the confirmed-positive seedset
(`data/seedset_raw.jsonl` / mirror `data/new_seedset.jsonl`) as the few-shot exemplar pool. Each near-miss
is rewritten under multiple `(round_idx, candidate_idx)` settings, the
candidate response is naturalness-ranked, and the highest-ranked
rewrites are kept. See `scripts/rebuild_near_misses.py`,
`scripts/rebuild_near_misses_v2.py`, `src/synthetic_generation/generate.py`,
and `scripts/filter_synthetic_by_similarity.py` for the current
implementation.

Output: `data/synthetic_data_full.jsonl` (raw rewrites; 1,722 rows in
the current run). A cosine-similarity filter on `(user_input,
source_input)` ≥ 0.75 narrows this to `data/synthetic_data.jsonl` (684
rows). The union relabel pass scores both seedset and synthetic rows
across all 9 measures using three responder models plus Opus 4.6 as judge
(see [`misc/relabel_synthetic.md`](misc/relabel_synthetic.md)). The
manual pass accepts 647 of the 684 relabelled synthetic rows and edits 14
accepted prompts, so the final synthetic inputs are derived from but not an
exact `user_input` subset of `data/synthetic_data_relabelled.jsonl`.

Final Stage 6 output and Stage 7 input:
`data/final_dataset.jsonl` — 969 rows, 3,147 tags, schema
`{user_input, measure, synthetic, language}`.

### Phase 3: Evaluation & Analysis

All three substages live under **Stage 7 — Evaluation**. Input is `data/final_dataset.jsonl` (969 rows: seedset + manual-pass synthetic). Keys are read from `.keys.json` at the repo root, with sections `{openrouter, deepseek, anthropic}` (gitignored); legacy `.openrouter_key` / `.deepseek_key` fallback files are also supported.

**Stage 7.1 — Generate Model Responses (OpenRouter/direct APIs + local HF Qwen):**
```bash
bash scripts/run_stage7_1_generate.sh
bash scripts/run_stage7_1_local_qwen.sh
# or directly:
uv run python -m src.evaluation.generate_responses \
    --input data/final_dataset.jsonl \
    --output data/eval_responses.jsonl \
    --keys .keys.json
```
Sends each `user_input` to **25 model evaluations** spanning 6 providers (Google, OpenAI, Anthropic, xAI, DeepSeek, Qwen). The first launcher runs the 23 API/direct entries in `generate_responses.py`; the second serves `qwen3_1_7b` and `qwen3_4b` locally from Hugging Face with vLLM, then merges those columns into the same `data/eval_responses.jsonl` schema.

The 25 evaluations break down as:
- **3 target** models: `gemini_2_flash_001`, `gpt_4o`, `claude_sonnet_4`
- **3 rewriting** models: `gemini_3_1_pro` (T), `gpt_5_4`, `claude_opus_4_6`
- **19 frontier** models: 16 distinct models + 3 thinking-budget variants of `claude-opus-4-6` (2k/5k/10k token budgets), each with their own column key.

DeepSeek models (`deepseek_chat`, `deepseek_v4_pro`, `deepseek_v4_flash`) call `https://api.deepseek.com/v1` directly using the `deepseek` key section; other API models route through OpenRouter. The Qwen models with active OpenRouter endpoints are `qwen3_6_max_preview`, `qwen3_8b`, `qwen3_14b`, and `qwen3_32b`. `qwen3_1_7b` and `qwen3_4b` are local-HF add-ons because OpenRouter currently lists them with 0 active endpoints. `qwen3_0_6b` remains excluded.

DSPy disk cache (`cache/dspy/`) keys on the full request payload — different thinking budgets land in distinct cache entries automatically. Use `--rollout_id N` to bypass the cache, `--models gpt_4o,deepseek_chat` to run a subset (useful for dry runs). The script hard-fails at startup on any missing required key section.

Total scale: 25 × 969 = **24,225 generations** (22,287 API/direct + 1,938 local HF).

**Stage 7.2 — LLM-as-Judge (OpenRouter, Opus 4.6 NT):**
```bash
bash scripts/run_stage7_2_judge.sh
# or directly:
uv run python -m src.evaluation.judge_responses \
    --input data/eval_responses.jsonl \
    --output data/eval_judge_results.jsonl \
    --keys .keys.json --concurrency 30
```
For every row, judges every model column against only that row's labelled `measure` list. Rows with multiple labels produce one judge call per (`user_input`, labelled measure, model) triple. Judge: Claude Opus 4.6 via OpenRouter, no thinking, temperature=0. Model columns are discovered dynamically from the input file's `model_responses` keys (no hardcoded model list).

Output: one JSONL row per (`user_input`, `measure`, `model_name`) triple with `judge_output: {reasoning, keep}`. Resumable: already-written triples are skipped before any LM call.

Total scale after the local-Qwen merge: 3,147 labelled input-measure pairs × 25 models = **78,675 judge calls**.

`sort_results.py` (optional post-step) sorts by input → measure → model family (OpenAI → Google → Anthropic → xAI → DeepSeek → Qwen, newest first).

**Stage 7.3 — Analysis:**
```bash
uv run python src/evaluation/analyze_results.py
```
Reads `data/eval_judge_results.jsonl` and produces figures + summary tables in `data/eval_figures/`. Family list and column display names live in the constants block at the top of the file — keep in sync with the API/direct registry plus local-HF Qwen add-on columns.

**Key sbatch design patterns (used in all pipeline scripts):**
- Port: `VLLM_PORT=$(python3 -c "import socket; s=socket.socket(); s.bind(('', 0)); print(s.getsockname()[1]); s.close()")` — OS-assigned free port, avoids conflicts
- TMPDIR: `job_${SLURM_JOB_ID}_${SLURM_ARRAY_TASK_ID}` — per-task to avoid torch inductor cache collisions on shared NFS
- `SLURM_ARRAY_TASK_COUNT` is set automatically by SLURM when using `--array=0-5` (no manual export needed)
- Checkpoint/resumption: filter stages skip rows already in the output file; safe to resubmit failed shards
- Snapshot: `--experiment_dir` in `src/filter/run.py` copies `src/filter/` into the experiment folder for reproducibility

## Project Layout
- `src/filter/` — filtering pipeline (Stages 1-4):
  - `run.py` — unified pipeline entry point; dispatches to `--measure` / `--stage`
  - `utils.py` — `Judge` base class + `JudgeConfig`; async inference, sharding, JSONL I/O
  - `param.py` — shared argparse definitions (`add_judge_args`)
  - `measure/` — one folder per measurement type; each is self-contained:
    - `<measure>/filter1.json` — coarse filter prompts (Stage 1)
    - `<measure>/filter2.json` — low-quality/domain-specific judge prompt (Stage 2)
    - `<measure>/filter3.json` — GPT-4o-mini dual-check config + system prompt (Stage 3)
    - `<measure>/filter4.json` — Claude Opus 4.6 config + system prompt (Stage 4)
    - Stage `.py` files (`coarse_filter.py`, `low_quality_filter.py`, `high_quality_filter.py`, `final_filter.py`) are provided by `measure/base/` and run automatically — **no need to add them per measure**
    - Current measures (9 total): `1B_intentional_human_speech/`, `1B_human_pronoun/`, `1C_identity_transparency/`, `2A_fabricated_personal_information/`, `2B_emotion_expression/`, `2C_deference/`, `2C_flattery_tone/`, `2D_human_relationship_encouragement/`, `3A_engagement_hooks/`
  - `mode/` — conversation formatting utilities:
    - `single_turn.py` — `format_single_turn(row)` for WildChat single-turn analysis
    - `multi_turn.py` — stub for future multi-turn support
- `src/data_preprocessing/` — Stage 5: data preprocessing (collect, dedupe, split)
  - `data_preprocessing_v2.py` — current v2 preprocessing record: curated cross-repo source list, both-KEEP collection, dedupe, and batched single/multi classification before manual seedset consolidation.
  - `data_preprocessing.py` — legacy v1 preprocessing script; useful as reference, but not the source of the current `final_dataset.jsonl`.
  - `intersect.py` — standalone utility to find intersection of `chitchat_keep` + `category_keep` from Stage 3 output
- `src/evaluation/` — Stage 7 evaluation pipeline (all substages):
  - `_eval_common.py` — shared constants (canonical 9 measures), judge-prompt loader, judge-output parser, `.keys.json` loader. Imported by every other file in this folder.
  - `generate_responses.py` — Stage 7.1 API/direct sweep: `MODEL_REGISTRY` of 23 model evaluations (OpenRouter + direct DeepSeek API + Anthropic thinking-budget variants). DSPy disk cache + output-file resumption. Default input: `data/final_dataset.jsonl`.
  - `generate_local_responses.py` — Stage 7.1 local-HF add-on for `qwen3_1_7b` and `qwen3_4b` served through vLLM on CARC GPUs. Writes API-compatible response part files.
  - `merge_local_responses.py` — merges local-HF response part files into `data/eval_responses.jsonl` without changing the Stage 7.1 output schema.
  - `judge_responses.py` — Stage 7.2: Opus-4.6 NT judges every (`user_input`, `model_col`, labelled `measure`) triple. Model columns discovered dynamically from the input file. Default input: `data/eval_responses.jsonl`.
  - `sort_results.py` — sort judge results by input → measure → model family/recency.
  - `analyze_results.py` — Stage 7.3: figures and summary tables from judge results.
  - `score_seedset.py` — one-shot scorer for the raw seedset (defaults to the mirror filename `data/new_seedset.jsonl`; use `--input data/seedset_raw.jsonl` locally). Shares helpers with the Stage 7 pipeline via `_eval_common`.
- `scripts/` — one-off Python scripts and stage launchers:
  - `download.py` — Stage 0: WildChat downloader.
  - `rebuild_near_misses_v2.py` — Stage 6: synthetic-generation pipeline (rewrite + naturalness rank).
  - `filter_synthetic_by_similarity.py` — Stage 6 post-filter: cosine-sim cut on `(user_input, source_input)`.
  - `run_stage7_1_generate.sh` — Stage 7.1 launcher for the 23 API/direct models on `final_dataset.jsonl`.
  - `run_stage7_1_local_qwen.sh` — Stage 7.1 local-HF launcher for `qwen3_1_7b` and `qwen3_4b`, followed by merge into `eval_responses.jsonl`.
  - `run_stage7_2_judge.sh` — Stage 7.2 launcher (Opus-4.6 NT judging on labelled measures).
- `experiments/` — auto-created by `pipeline.sh`; one folder per stage run (`<NN>_<stage>/`)
  - Each folder contains `figures/`, `logs/`, `results/`, and an `src/` snapshot (shard 0 only)
- `data/` — datasets (gitignored)
  - `wildchat_raw.jsonl` — downloaded WildChat data (Stage 0)
  - `seedset_raw.jsonl` — current raw in-the-wild seedset, 322 rows / 368 original tags.
  - `seedset_data.jsonl` — relabelled in-the-wild seedset, 322 rows / 960 tags.
  - `near_misses.jsonl` — Stage 6 rewrite source pool, 11,520 rows.
  - `synthetic_data_full.jsonl`, `synthetic_data.jsonl`, `synthetic_data_relabelled.jsonl` — Stage 6 synthetic generation outputs (raw, cos-sim filtered, and union-relabelled).
  - `final_dataset.jsonl` — Stage 6 final output and **Stage 7.1 input**: seedset + manual-pass synthetic, 969 rows, schema `{user_input, measure, synthetic, language}`.
  - `eval_responses.jsonl` — Stage 7.1 output: 25-model responses per `user_input` after the local-Qwen merge.
  - `eval_judge_results.jsonl` — Stage 7.2 output: one row per (input, labelled measure, model) triple, judged by Opus-4.6 NT.
- `slurm/` — generic SLURM job templates (for simple single-script jobs)
- `tests/` — pytest tests
- `depreciated/` — old monolithic pipeline scripts (reference only, do not edit)

## Conventions
- **Adding a new measure:** create `src/filter/measure/<name>/` with `filter1.json`, `filter2.json`, `filter3.json`, and `filter4.json`. Then run `bash pipeline.sh --measure <name> --stage <stage>` — the base implementations handle everything automatically.
- **Experiments:** `experiments/` starts empty and is populated automatically by `pipeline.sh`. Each run creates a new `<NN>_<stage>/` folder. Do not manually add files there.
- Pipeline stages with vLLM use `slurm/run_vllm_stage.sbatch` (parameterized via env vars set by `pipeline.sh`).
- The top-level `experiments/README.md` should only contain brief descriptions of each experiment.
- Reusable code goes in `src/` with argparse for flexibility.
- Tracking: wandb (disable with `WANDB_MODE=disabled`)
- **Data format:** After Stage 5, `measure` is always a list (e.g., `["2C_flattery_tone"]` or `["1B_intentional_human_speech", "3A_engagement_hooks"]`). Conversations are deduplicated by `user_input` — a conversation flagged for multiple categories appears once with all measures listed.

## Paper Drafting Notes

The paper repo under `paper/` is currently outdated relative to the data
pipeline. For rewriting Section 3 (`paper/03_dataset.tex`), treat
`misc/data_summary.md` and `misc/data_results.md` as the source of truth.
`misc/section3_rewrite_system_prompt.md` contains the browser-model prompt
for that rewrite. Do not copy dataset counts or model-result claims from
the current abstract, results, or discussion until those sections are
refreshed.

## Programming Philosophy

The purpose of all code is to serve as educational material for the *student reader*. The student reader is someone who is reading the code for the sake of learning the concept implemented therin. To aid in teaching the student reader, all code should be simple, straightforward, and structured so as be as easy as possible to parse and understand on all levels. To meet this pedagogical goal, adhere to the following guidelines:

### Focus on the core idea

All code should clearly contribute to the core idea of the program. Exclude all code which is tangential or orthogonal to the core idea. For example, if you are writing a program to calculate the factorial of a user input, focus on writing the factorial function itself above all else. Do not include code which deviates from the core idea to do things such as sanitize inputs, annotate types, catch edge case errors, etc. This is an opionated take, but it is an important part of my focus on simplicity and core ideas. The only exception to this is type annotations in Pydantic classes, which absolutely should be included when Pydantic expects it. Read more about Pydantic in the Correctness section later.

### Name variables well

It should be clear at a glance what the purpose of a variable is. The student reader should be able to follow the code easily, which means variable names should not include abbreviations but should also not be excessively long either. When in doubt, think about what is easiest on the eyes and brain of the student reader.

### Use comments for the benefit of the student reader

As the student reader reads the code, there is lots of information that would help their learning process that isn't immediately apparent from the code itself. This is where comments come in. The following are the main categories of comments to include:

### Performance improvements

High performance code can sometimes be complex and difficult to parse, which goes against the ideal of teaching the student reader. When you encounter this situation, first think about how you can write code which is both elegant and performant. But in cases where there is a true, anavoidable tradeoff, prefer the simplest approach but note that this code has room for performance improvements and how to go about implementing them. These comments need not be long explanations, just enough to teach the student reader something about where they can go next.

**format:** `NOTE: [performance improvement] <content>`

### Thought process explanations

The code you write will be a result of a detailed thought process which is not self evident to the reader. Whenever this thought process presents a learning opportunity, take advantage of it and explain to the student reader why somthing is the way it is. They may not have thought about things this way before, or they may have a different opinion about what to do. Regardless, the reader can benefit from certain non-obvious insights into the decision making process.

**format:** `NOTE: [thought process] <content>`

### Pedagogical information

This is a core avenue for teaching the student reader. When there are opportunities to use advanced techniques to make code better in some way (often simpler, less memory-intensive, etc.), use these techniques and call them out to the student reader. In addition, there might be non-obvious side effects or mathematical insights behind certain pieces of code. Whenever it may benefit the student reader and it isn't clear from the code, call out pedagogical insights explicitly with a comment.

Here's a great example of this from a python program:

\# NOTE: [pedagogical] CrossEntropyLoss combines log-softmax and negative log-likelihood
\# in one step. It expects raw logits (not softmax outputs), which is why the model's
\# final layer has no activation function.

**format:** `NOTE: [pedagogical] <content>`

### Shape explanations

The shape of tensors is of importance to the ML code you will be writing. If at any point you reference the shape of a tensor, make a callout explaining what concept this refers to. For example, .size(0) might represent the batch size, and it's important to explain this with a comment. Additionally, any time broadcasting is occuring it is important to explain the two shapes and how the broadcast happens. \*, @, and other possibly confusing shape-involved operations should be explained too.

Examples:

`NOTE: [shape] taking batch from batch x sequence_len x model_dimNOTE: [shape] t1: batch x sequence_len x model_dim @ t2: model_dim. @ in pytorch does matrix multiplication; this operations works like <provide explanation>`

**format:** `NOTE: [shape] <content>`

### Flagging possible edge cases

As mentioned before, it's important not to bog down code with edge case error handling because it distracts from the core idea. However, these edge cases can still be important. To call attention to these for teaching purposes and possible future handling, add a comment explaining what they are.

**format:** `NOTE: [edge case callout] <content>`

### Optimize the layout of code for the student reader

The layout of code should be simple to follow above all else. This means the control flow should be easy to follow, but also the code should be easy to follow with the eye. For example, a function which only runs once during initialization could be written out inside the main function instead of factored out into its own function. This isn't always obvious, so always use your best judgement to determine what would most enhance the learning experience of the student reader. Just keep in mind that function calls can be opaque.

### Development Progression

Implement the simplest approach first. Complexity can be built up over time, but it should only be done as necessary over the simpler approaches. Consider adding comments explaining ideas for future expansion instead of adding complexity when it is not yet necessary to do so.

## Correctness

The work that I do has zero tolerance for bugs in final code. I'm doing ML research, so a bug in the code can completely ruin a paper by giving misleading results. The solution to this is not to avoid writing bugs in the first place but rather to rigourously test and verify code. Use software tools to increase confidence in the code, not proofreading. To do this, adhere to the following:

### Code structure

Code should be written so that adding instrumentation is easy and clear, not hard and all over the place. This is done by preferring simple control flow and keeping in mind what print statements might be useful. I may ask to add print statements, assertions, and other small checks inline with the code. This shouldn't affect code layout a ton, but just keep it in mind whenever there are clear layout wins that aid potential instrumentation.

---
> Source: [ehuang996/socialai](https://github.com/ehuang996/socialai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
