## galaxy-benchmark

> Execute Galaxy-Bench runs with immutable trace capture, explicit attempt-by-attempt planning, Galaxy provenance preservation, downloaded result artifacts, and structured evaluation outputs.


# Galaxy-Bench Executor

Use this skill when executing benchmark experiments in this repository.

## 1. Core Principle

A run is benchmark-valid only if a third party can reconstruct, from saved artifacts:

- what the agent planned before each attempt
- what the agent reasoned and decided during execution
- what Galaxy executed
- which tools, parameters, preprocessing steps, and outputs were used
- what failed, how the failure was interpreted, and how it was fixed
- what final outputs were produced
- how those outputs were evaluated

If evidence is not stored in artifacts, it does not count.

## 2. Write Boundary

During benchmark execution, write only under the run directory:

- `outputs/<timestamp>_<level>_<experiment>/`

Do not write benchmark-execution artifacts anywhere else. Optional extra artifacts are allowed only inside the run directory.

## 3. Tools Boundary

Galaxy is the required execution environment for Galaxy-mediated benchmark runs.

The agent is not allowed to use Galaxy Interactive Tools at all.

The use of AWK is not allowed unless the prompt explicitly states that AWK may be used.

Local commands may be used only to support the benchmark contract, such as inspecting allowed task metadata, preparing files for Galaxy upload, preserving downloaded Galaxy outputs, transforming already-produced Galaxy outputs when allowed by this skill, reproducing the recorded workflow, and evaluating fixed outputs after the ground-truth access rules allow it.

Do not perform the scientific analysis locally as the primary execution path for Galaxy runs or BixBench runs.

## 4. Run Directory Contract

Every run must create this directory shape:

```text
outputs/<timestamp>_<level>_<experiment>/
|-- experiment_summary.json
|-- plan/
|   |-- saved.md
|   `-- saved.attempt_<N>.md
|-- reasoning/
|   |-- reasoning.md
|   `-- reasoning.attempt_<N>.md
|-- errors/
|   `-- error.json
|-- traces/
|   |-- galaxy/
|   |   |-- histories/
|   |   |-- invocations/
|   |   |-- jobs/
|   |   `-- datasets/
|   `-- local/
|-- evaluations/
|   |-- comparison.json
|   |-- comparison.attempt_<N>.json
|   `-- metrics_summary.json
`-- results/
    |-- result.json
    |-- result.attempt_<N>.json
    |-- activity_log.jsonl
    |-- run_record.json
    |-- artifacts_manifest.json
    |-- evaluation_manifest.json
    `-- reproduce_<experiment>.py
```

Attempt-specific files are required when retries or separately evaluated attempts occur.

## 5. Artifact Requirements

### Required Final Files

A completed benchmark-valid run must include:

- `experiment_summary.json`
- `plan/saved.md`
- `reasoning/reasoning.md`
- `errors/error.json`
- `results/result.json`
- `results/reproduce_<experiment>.py`
- `results/run_record.json`
- `results/artifacts_manifest.json`
- `results/evaluation_manifest.json`
- `results/activity_log.jsonl`
- `evaluations/comparison.json`
- `evaluations/metrics_summary.json`
- original downloaded Galaxy output files used for evaluation, preserved unchanged

Before declaring completion, verify that:

- required outputs exist
- `experiment_summary.json` points to the ground truth, Galaxy tools, original Galaxy result files, transformed outputs, and final scores
- manifests reference all preserved artifacts
- the activity log covers planning, execution, checks, retries, revisions, snapshots, and evaluation
- Galaxy IDs referenced in results are preserved in traces
- `errors/error.json` has a terminal status
- no prior attempt artifact was overwritten
- evaluation JSON explicitly shows the required metrics

### `experiment_summary.json`

Every non-BixBench run must write and maintain a root-level summary:

- `outputs/<timestamp>_<level>_<experiment>/experiment_summary.json`

This is the reviewer-facing index for the run. It does not replace detailed evidence in `results/`, `evaluations/`, or `traces/`.

Required non-BixBench shape:

```json
{
  "experiment": "<experiment_name>",
  "Ground_truth_path": [
    "<path to each ground truth file used for comparison>"
  ],
  "Galaxy_tools_used": [
    "<Galaxy tool id or display name used for this experiment>"
  ],
  "Galaxy_results": {
    "files": [
      "<Galaxy file name or HID/name considered a final result for the task>"
    ],
    "path": [
      "<path to the preserved result file in this run directory>"
    ]
  },
  "Transformed_galaxy_output": [
    "<path to each transformed Galaxy-derived output used for comparison>"
  ],
  "Experiment_score": {
    "prompt_score": 0.0,
    "transformed_prompt_score": 0.0,
    "direct_ground_truth_match_score": 0.0,
    "transformed_ground_truth_match_score": 0.0,
    "agent_performance_in_galaxy_score": 0.0
  },
  "Evaluation_questions": {
    "prompt_requirements": {
      "question": "Does the Galaxy output satisfy the requirements from the prompt?",
      "answer": "<yes/no/partial>",
      "score": 0.0,
      "matched_requirements": 0,
      "total_requirements": 0,
      "basis": [
        "<short evidence statement>"
      ]
    },
    "transformed_prompt_requirements": {
      "question": "Does the agent-rearranged Galaxy output satisfy the requirements from the prompt?",
      "answer": "<yes/no/partial/not_available>",
      "score": 0.0,
      "matched_requirements": 0,
      "total_requirements": 0,
      "basis": [
        "<short evidence statement>"
      ]
    },
    "direct_ground_truth_match": {
      "question": "Does the original Galaxy output directly match the ground truth?",
      "answer": "<yes/no/partial/not_available>",
      "score": 0.0,
      "matched_items": 0,
      "compared_items": 0,
      "match_percent": 0.0,
      "basis": [
        "<short evidence statement>"
      ]
    },
    "transformed_ground_truth_match": {
      "question": "Does the agent-rearranged Galaxy output match the ground truth?",
      "answer": "<yes/no/partial/not_available>",
      "score": 0.0,
      "matched_items": 0,
      "compared_items": 0,
      "match_percent": 0.0,
      "basis": [
        "<short evidence statement>"
      ]
    },
    "agent_execution": {
      "question": "Does the agent know how to execute the task in Galaxy to reach the result?",
      "answer": "<yes/no/partial>",
      "score": 0.0,
      "failure_count": 0,
      "required_output_achieved": true,
      "basis": [
        "<short evidence statement>"
      ]
    }
  }
}
```

Field rules:

- `Ground_truth_path` lists only ground-truth/reference files actually used by the evaluator.
- `Galaxy_tools_used` lists Galaxy tools that materially contributed to the completed task, including tool IDs when available.
- `Galaxy_results.files` identifies the Galaxy history outputs considered final task results, preferably with HIDs or dataset IDs.
- `Galaxy_results.path` lists local preserved copies of those Galaxy outputs.
- `Transformed_galaxy_output` lists only transformed helper files used for scoring or comparison; use an empty list if none were used.
- `Evaluation_questions` answers the benchmark questions in reader-facing language and includes relevant counts whenever available.
- Paths should be relative to the run directory unless an external reference path is required for auditability.

Score mapping:

- `prompt_score` maps to prompt-requirement compliance for original Galaxy outputs.
- `transformed_prompt_score` maps to prompt-requirement compliance after reshaping preserved Galaxy outputs.
- `direct_ground_truth_match_score` maps to direct item-level comparison between original Galaxy output files and ground truth.
- `transformed_ground_truth_match_score` maps to item-level comparison between agent-rearranged Galaxy-derived output and ground truth.
- `agent_performance_in_galaxy_score` maps to the agent's execution score in Galaxy.
- Use `null` only when a direct or transformed ground-truth comparison is not meaningful or not performed, and explain why in `Evaluation_questions`.
- Prompt-score evidence must not be treated as ground-truth-match evidence.

### `plan/`

Before running the experiment in Galaxy, write `plan/saved.md` with:

- experiment objective
- input datasets
- intended workflow steps
- intended tool choices
- expected result files
- anticipated risks and fallback branches

For each retry, create `plan/saved.attempt_<N>.md` describing what changed and why the new attempt is being launched.

### `reasoning/`

Record decision-making during execution, not reconstructed afterward. Include:

- tools considered, selected, and rejected
- parameters considered and selected
- preprocessing decisions
- assumptions, uncertainty, and confidence estimate or proxy before major milestones
- why a plan was chosen
- errors encountered and interpreted
- root-cause analysis and fix strategy before each retry
- changes between attempts
- retries and stopping decisions

`reasoning/reasoning.md` is required. `reasoning/reasoning.attempt_<N>.md` is optional when an attempt-specific record is useful.

### `errors/`

`errors/error.json` must preserve the full error history. For each error, record:

- source
- failed tool or workflow step
- stable error signature
- message
- timestamp
- whether it was fixed
- what changed in response

Never blind-retry. For every failed attempt:

1. Snapshot failure evidence.
2. Extract a stable error signature.
3. Record the inferred root cause.
4. Record the fix strategy.
5. Log a `revise` record before retrying.
6. Write new attempt artifacts.

If the same signature repeats, change the failing mechanism or stop with a documented blocker.

### `traces/`

Preserve structured Galaxy evidence under `traces/galaxy/` whenever available:

- history names and IDs
- dataset names, IDs, HIDs, metadata, and states
- job IDs, states, parameters, provenance, stdout, and stderr
- invocation IDs and workflow IDs
- tool IDs
- state transitions and polling results
- Galaxy metadata snapshots

If a Galaxy object is important to execution, scoring, or debugging, snapshot it.

Preserve local execution evidence under `traces/local/` or `results/`, including a command log or script log updated as the agent executes. `results/reproduce_<experiment>.py` remains the canonical replay script.

Polling and wait policy:

- When a Galaxy job or output dataset is active, wait for completion unless there is a clear terminal failure signature.
- Start with `1 minute` between checks.
- Keep the `1 minute` cadence until `6 minutes` total wait time has elapsed.
- After `6 minutes`, increase to `15 minutes` between checks.
- At each check, preserve timestamp, observed state, latest relevant `update_time`, and any newly available stdout/stderr or failure details.
- Stop early only for terminal error state, a stable repeated blocker with preserved evidence, or documented resumption from an already preserved Galaxy state.

### `results/`

Store:

- final structured result in `results/result.json`
- attempt-specific result versions as `results/result.attempt_<N>.json`
- downloaded Galaxy result files
- supporting result artifacts
- output formats produced
- `results/run_record.json`
- `results/artifacts_manifest.json`
- `results/evaluation_manifest.json`
- append-only `results/activity_log.jsonl`
- canonical replay script `results/reproduce_<experiment>.py`

Append JSONL activity records for `plan`, `execute`, `check`, `retry`, `revise`, `evaluate`, and `snapshot`. Each record should include relevant IDs, paths, parameters, and outcomes.

### `evaluations/`

After result generation, write:

- `evaluations/comparison.json`: detailed evaluation evidence, scoring logic, counts, and calculation notes
- `evaluations/comparison.attempt_<N>.json`: attempt-specific comparisons when multiple attempts are evaluated
- `evaluations/metrics_summary.json`: concise final metrics only

For non-BixBench runs, `evaluations/comparison.json` should include:

- `prompt_result_evaluation`
- `transformed_prompt_result_evaluation`
- `direct_ground_truth_result_evaluation`
- `transformed_ground_truth_result_evaluation`
- `agent_performance_in_galaxy_score`
- `calculation_notes`

For non-BixBench runs, `evaluations/metrics_summary.json` must include only:

- `prompt_result_score`
- `transformed_prompt_result_score`
- `direct_ground_truth_result_score`
- `transformed_ground_truth_result_score`
- `agent_performance_in_galaxy_score`

Do not include helper metrics, overlap counts, revision markers, timestamps, explanatory text, or the legacy label `galaxy_performance_score`.

#### Evaluation Metric Rules

`prompt_result_evaluation` answers whether original Galaxy-produced outputs meet prompt requirements, such as required format, headers, output fields, and deliverable type. Score by `matched_requirements / checked_requirements`. Use the ground-truth `expected_outputs` text only to understand the expected result contract; do not treat prompt compliance as ground-truth matching.

`transformed_prompt_result_evaluation` answers whether agent-rearranged Galaxy output meets prompt requirements. It is still prompt compliance, not ground-truth matching, and it does not replace original-output prompt scoring. Preserve original Galaxy outputs unchanged; store transformed files separately; derive transformed files only from the original downloaded Galaxy outputs chosen as evaluation targets.

`direct_ground_truth_result_evaluation` compares the original downloaded Galaxy output files directly to ground truth. Count comparable reference elements and item-level matches. Do not modify, normalize, regenerate, rewrite, or synthesize substitutes for this direct score. If direct comparison is not meaningful because formats are incompatible, record that and use `null`.

`transformed_ground_truth_result_evaluation` compares a transformed Galaxy-derived output to ground truth. It exists only to remove output-format incompatibility when the scientific result is already present in the Galaxy output.

Allowed transformed-output operations:

- renaming columns
- reordering rows or columns
- selecting subsets of rows or columns
- delimiter changes
- file-format conversion
- deterministic joins across original Galaxy outputs when all scored values already exist in those outputs

Disallowed transformed-output operations:

- adding inferred or externally sourced annotations
- recomputing scientific results outside the original Galaxy output content
- filling missing values with guessed, inferred, or newly derived content
- normalizing values through external mappings not already present in the original Galaxy output
- creating scored values not directly traceable to original Galaxy output files
- forcing output to match ground truth by introducing information Galaxy did not produce

For transformed ground-truth scoring:

- list exact original Galaxy source files
- state whether the transformation is `format-only` or `content-altering`
- explain transformation steps
- ensure every scored value is traceable to original Galaxy output values
- mark the metric invalid or not eligible if content-altering transformation introduced scored values

`agent_performance_in_galaxy_score` answers whether the agent knew how to execute the task in Galaxy to reach the result. Scoring rule:

- start at `100`
- deduct `50` if the required output was not achieved
- deduct `10` for each failure
- score `0` if the run fails completely with no output and no completed steps

Make the calculation explicit in `evaluations/comparison.json`.

## 6. Immutability Rules

- `plan/saved.md` is the initial plan and must not be overwritten.
- Every retry must create new attempt-specific plan, result, and comparison artifacts.
- `reasoning/reasoning.md` is append-only.
- `results/activity_log.jsonl` is append-only.
- Do not replace or delete prior result, trace, or evaluation artifacts.
- If correction is needed, write a new versioned artifact and update manifests.
- Preserve Galaxy evidence snapshots and IDs.

## 7. Credential Gate

Before any Galaxy action:

- verify `.env` contains `GALAXY_API_KEY`
- stop if missing or empty

Do not print, log, or persist secret values.

## 8. BixBench Evaluation Rules

BixBench tasks are final-answer benchmarks, not workflow-quality benchmarks. Use this section for:

- `experiments/BixBench/task_<N>.json`

The local task set is the 50-question verified subset from `phylobio/BixBench-Verified-50`. Matching ground truth lives under:

- `ground_truth/BixBench/task_<N>.json`

### Required Galaxy Environment

All BixBench task analyses must be performed in Galaxy Project at `usegalaxy.org`.

Do not perform the scientific analysis locally as the primary execution path. Local commands may only be used to:

- inspect task metadata and allowed input file schemas before upload
- prepare files for upload to Galaxy
- preserve downloaded Galaxy outputs and trace artifacts
- transform already-produced Galaxy outputs into the submitted-answer shape when allowed by this skill
- evaluate the fixed submitted answer after the ground-truth access gate opens

The submitted answer must be traceable to outputs generated through `usegalaxy.org`. Preserve Galaxy history, dataset, job, and output evidence under `traces/galaxy/`, and record Galaxy-derived files used to decide the answer under `results/`.

### Ground-Truth Access Gate

For BixBench, the ground-truth file is hidden during task execution. The agent must not open, read, search, summarize, or otherwise use:

- `ground_truth/BixBench/task_<N>.json`
- any other task-related ground-truth file

until all of the following are true:

1. The agent completed the analysis using only the prompt and allowed input files.
2. The agent wrote the final submitted answer into run artifacts.
3. The agent recorded that final answer as if calling `submit_answer(answer="<final answer>")`.

Only after this final answer is fixed may the evaluator open ground truth to score the run. Do not revise the submitted answer after opening ground truth.

### Required BixBench Result

Before evaluation, `results/result.json` must include:

```json
{
  "experiment": "BixBench/task_<N>",
  "submitted_answer": "<final answer>",
  "submit_answer_called": true,
  "status": "submitted"
}
```

The `submitted_answer` is the only scientific output BixBench grades. Preserve supporting analysis files separately, but do not use them as substitutes for the submitted answer.

### Scoring Model

BixBench scoring is binary:

- `1.0` if the submitted answer is accepted
- `0.0` if the submitted answer is not accepted

BixBench does not grade intermediate reasoning, tool choice, code quality, preprocessing count, workflow elegance, or partial scientific credit. Preserve those details for auditability, but do not let them change the BixBench correctness score.

Use `agent_performance_in_galaxy_score` only to report whether the agent successfully executed the required environment workflow. It must not override or inflate the binary answer score.

### Verifier Behavior

The ground-truth file contains evaluator-only fields including `ideal`, and may include `distractors`, `hypothesis`, `eval_mode`, and related metadata.

For `eval_mode: "str_verifier"`:

1. Compare `submitted_answer` with `ideal` exactly, ignoring case.
2. If that fails, use a semantic-equivalence grader to decide whether the submitted answer means the same thing as the ideal answer.
3. The current BixBench semantic grader is `gpt-5-mini`.

For `eval_mode: "range_verifier"`:

1. Parse `submitted_answer` and `ideal` as numeric values or numeric ranges.
2. If no numeric distractors are available, accept answers within `1%` relative tolerance of the ideal value.
3. If numeric distractors are available, accept only when the submitted value is closer to the ideal than to every distractor.

If parsing fails or the answer is not semantically or numerically acceptable, score `0.0`.

### BixBench Evaluation Artifacts

After opening ground truth, write `evaluations/comparison.json`:

```json
{
  "bixbench_answer_evaluation": {
    "submitted_answer": "<final answer>",
    "ideal": "<hidden ideal read after submission>",
    "eval_mode": "str_verifier",
    "accepted": true,
    "score": 1.0,
    "basis": [
      "Case-insensitive exact match to ideal answer."
    ]
  },
  "ground_truth_access": {
    "opened_after_submit_answer": true,
    "ground_truth_path": "ground_truth/BixBench/task_<N>.json"
  }
}
```

For BixBench, `evaluations/metrics_summary.json` must contain only:

```json
{
  "bixbench_answer_score": 1.0,
  "agent_performance_in_galaxy_score": 100.0
}
```

### BixBench `experiment_summary.json`

For BixBench runs, use this reduced shape:

```json
{
  "experiment": "bixbench_task_<N>",
  "Ground_truth_path": [
    "ground_truth/BixBench/task_<N>.json"
  ],
  "Galaxy_tools_used": [
    "<Galaxy tool id or display name used for this experiment>"
  ],
  "Galaxy_results": {
    "files": [
      "<Galaxy file name or HID/name considered a final result for the task>"
    ],
    "path": [
      "<path to the preserved result file in this run directory>"
    ]
  },
  "Experiment_score": {
    "ideal": "<value from ground truth>",
    "Galaxy_answer": "<value extracted from Galaxy analysis>",
    "direct_ground_truth_match_score": 1.0
  }
}
```

BixBench summary rules:

- Keep only `experiment`, `Ground_truth_path`, `Galaxy_tools_used`, `Galaxy_results`, and `Experiment_score`.
- Do not include `Transformed_galaxy_output` or `Evaluation_questions`.
- Do not include non-BixBench prompt, transformed-prompt, transformed-ground-truth, or agent-execution score fields inside `Experiment_score`.
- Copy `Experiment_score.ideal` from hidden ground truth only after the ground-truth access gate opens.
- Set `Experiment_score.Galaxy_answer` to the fixed submitted answer extracted from preserved Galaxy analysis outputs before ground-truth access.
- Set `Experiment_score.direct_ground_truth_match_score` equal to the binary answer score from `evaluations/comparison.json`.
- Before evaluation, `Ground_truth_path` must be an empty list; after evaluation, list `ground_truth/BixBench/task_<N>.json`.

The BixBench pipeline is:

1. Agent receives question and files.
2. Agent executes the analysis through `usegalaxy.org` without ground-truth access.
3. Agent downloads and preserves Galaxy outputs used to decide the answer.
4. Agent fixes a final answer via `submit_answer(answer="...")` and records it.
5. Evaluator opens ground truth.
6. Evaluator compares the submitted answer with hidden `ideal` using the declared verifier.
7. Score is `1.0` if accepted, otherwise `0.0`.

---
> Source: [paulocilasjr/Galaxy_benchmark](https://github.com/paulocilasjr/Galaxy_benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
