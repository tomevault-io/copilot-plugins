## autoeoh

> AutoEoH is a natural-language orchestrated workflow for automated heuristic

# AutoEoH Agent Instructions

AutoEoH is a natural-language orchestrated workflow for automated heuristic
algorithm design. Use these instructions in any agent environment unless a
tool-specific adapter overrides only the entry wording.

## Objective

Given either:

- a text problem description,
- an existing solver/codebase,
- a saved task package, or
- an existing EoH run directory,

execute the appropriate AutoEoH workflow and leave the repository in a state
where outputs are written to disk and easy to inspect.

## Source of Truth

The execution contract is defined by:

- `agents/task_creator/agent.md`
- `agents/codebase_analyzer/agent.md`
- `agents/task_formulator/agent.md`
- `agents/task_checker/agent.md`
- `agents/evaluation_checker/agent.md`
- `agents/eoh_runner/agent.md`
- `agents/results_checker/agent.md`
- `shared/config_loader.py`
- `shared/task_package.py`

If those files disagree, prefer the implemented Python modules over prose.

## Pipeline Modes

Choose one entry mode based on what you have available:

| Mode | When to use |
|---|---|
| `from_description` | Start from a text problem description |
| `from_code` | Start from existing algorithm code |
| `from_task` | TaskPackage already exists on disk |
| `check_only` | Re-check an existing run directory |
| `eval_check_only` | Diagnose null-score issues without a full run |

### Data Flow

```
config.yaml
    -> build_llm()
    -> TaskCreatorAgent  (from_description)
       CodebaseAnalyzeAgent -> TaskFormulatorAgent  (from_code)
       TaskPackage.load()  (from_task / check_only / eval_check_only)
    -> TaskCheckerAgent
    -> EvaluationCheckerAgent
    -> EoHRunnerAgent
    -> ResultsCheckerAgent
    -> Run summary
```

## Required Workflow

### Mode 1 — `from_description`

> Start from a `description.md` or user-provided text.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `TaskCreator` → writes task package to disk
- [ ] Step 3 — Run `TaskChecker` → validates task package
- [ ] Step 4 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 5 — Run `EoHRunner` → executes evolution
- [ ] Step 6 — Run `ResultsChecker` → summarises best result

### Mode 2 — `from_code`

> Read the target codebase and optional dataset path.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `CodebaseAnalyzer` → identifies the key heuristic component
  - `TaskFormulatorAgent` auto-loads `description.md` from the example folder
    (`code_dir` or its parent) and passes `Problem Statement` + `Heuristic Goal`
    as `problem_description` to both `CodebaseAnalyzer` and `TaskFormulator`.
- [ ] Step 3 — Run `TaskFormulator` → writes task package to disk using that analysis
- [ ] Step 4 — Run `TaskChecker` → validates task package
- [ ] Step 5 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 6 — Run `EoHRunner` → executes evolution
- [ ] Step 7 — Run `ResultsChecker` → summarises best result

### Mode 3 — `from_task`

> Load an existing `TaskPackage` from disk.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Load `TaskPackage` from disk
- [ ] Step 3 — Run `TaskChecker` if package was newly generated or edited
- [ ] Step 4 — Run `EvaluationChecker` → validates evaluation function
- [ ] Step 5 — Run `EoHRunner` → executes evolution
- [ ] Step 6 — Run `ResultsChecker` → summarises best result

### Mode 4 — `check_only`

> Re-check results of a completed run without re-running EoH.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Run `ResultsChecker` on the existing run directory

### Mode 5 — `eval_check_only`

> Diagnose null-score issues without starting a full EoH run.

- [ ] Step 1 — Load `config.yaml` and build LLM backend
- [ ] Step 2 — Load `TaskPackage` from disk
- [ ] Step 3 — Run `EvaluationChecker` to diagnose the evaluation function

Do not skip `TaskChecker` for newly created tasks.

## Configuration

Load configuration from `config.yaml` using `shared.config_loader.load_config()`.
Build the LLM backend using `shared.config_loader.build_llm()`.

Provider-neutral environment variable precedence:

1. `AUTOEOH_LLM_API_KEY`
2. Provider fallback such as `OPENAI_API_KEY` or `ANTHROPIC_API_KEY`

## Agent Expectations

- Prefer deterministic file outputs over conversational summaries.
- Save generated tasks via `TaskPackage.save(directory, config=config)`.
  Always pass the current `config` dict so that `runEoH.py` and
  `run_pipeline.py` are generated with all LLM credentials and EoH parameters
  embedded.
- `TaskPackage.save()` writes two user-facing scripts:
  - `<task_dir>/runEoH.py` — self-contained EoH-only re-run entry point.
  - `<task_dir>/../run_pipeline.py` — full pipeline entry point (TaskChecker
    → EvaluationChecker → EoHRunner → ResultsChecker) written one level above
    the task directory, alongside the `output/` folder.  Users run
    `python run_pipeline.py` from the example directory to execute the entire
    pipeline in a single process without any additional authorisation steps.
- Keep task metadata populated with provenance where available.
- When a documented path in prose does not exist, inspect the actual repository
  and use the real path instead of assuming the docs are current.
- When a configuration option is documented but unimplemented, fail clearly or
  degrade explicitly; do not silently pretend support exists.

## Tooling Notes

- `shared/anthropic_api.py` is the Anthropic backend.
- `shared/claude_api.py` is a backward-compatible alias.
- For non-Anthropic providers, use the OpenAI-compatible EoH HTTPS backend.

## Expected User Requests

Examples of supported natural-language requests:

- "Create a task from `examples/07_optuna/description.md` and run EoH."
- "Formulate a task from the code in `examples/02_deap`."
- "Validate the generated task before running."
- "Re-check the run in `examples/04_pygmo/output/run`."

## Pipeline Status Protocol

Long-running evolution sessions may survive context compaction or session
restarts.  To allow any new session to resume without re-reading all logs, each
agent **must** maintain a YAML status file at `shared/pipeline_status.yaml`.

### Format

Write or overwrite `shared/pipeline_status.yaml` with this structure:

```yaml
mode: from_code
stage: task_checker
problem: pymoo_moead
task_dir: examples/01_pymoo/output/task
run_dir: examples/01_pymoo/output/run
generation: null
best_score: null
error: null
next: evaluation_checker
updated: 2026-03-25T15:33:30
```

### Rules

1. **Write on every agent transition.**  Before calling the next agent, update
   `stage`, `next`, `best_score`, and `updated`.
2. **Write on error.**  Set `error` to a one-line summary and `next` to the
   recommended recovery action (e.g. `retry task_checker`, `manual_fix`).
3. **Read on session start.**  When a new session begins or context compacts,
   read `shared/pipeline_status.yaml`.  If present and `updated` is less than
   24 hours old, resume from `next` without asking the user.  If stale (>24 h),
   confirm with the user before resuming.
4. **Clear on completion.**  After `ResultsChecker` finishes, overwrite with a
   completion summary:
   ```yaml
   mode: from_description
   stage: completed
   problem: moead_leader_selection
   best_score: 0.635
   run_dir: examples/01_pymoo/output/run
   summary: examples/01_pymoo/output/run/results_summary.md
   updated: 2026-03-25T14:30:00
   ```
5. **One pipeline at a time.**  Starting a new pipeline overwrites the previous
   status file.

### Recovery Behaviour

On session start or post-compaction, the agent should:

1. Read `shared/pipeline_status.yaml`.
2. If `stage: completed` → inform user of prior results; await new request.
3. If `stage` is an agent name and `updated` is fresh → resume from `next`.
4. If `error` is non-null → report error and ask user how to proceed.
5. If file does not exist → start fresh; await user request.

## Terminal Output Style

### Response brevity

- **≤3 sentences of prose per response.** Lead with the action taken or decision made — never preamble, restatements, or pleasantries.
- **One line per agent action.** Use `[AgentName] verb noun.` format.
- **No filler.** Omit "Starting…", "Please wait…", "Done!" — print only what happened.
- **Errors on one line.** `[AgentName] ERROR: <message>`, exception on the next line only.

### Pipeline todo list

**When a pipeline is active, print this block at the top of every response** (read state from `shared/pipeline_status.yaml`):

```
[ AutoEoH · <mode> · <problem> ]
  [x] codebase_analyzer
  [x] task_formulator
  [ ] task_checker          ← next
  [ ] evaluation_checker
  [ ] eoh_runner
  [ ] results_checker
```

Rules:
- `[x]` completed, `[ ]` pending, `← next` on the stage about to run.
- Omit stages not in the current mode.
- Omit the block entirely when no pipeline is active.

Each agent also prints this same header as its **first terminal line** before any other output:

```
━━━ AutoEoH · from_code · pymoo_moead ━━━━━━━━━━━━━━━━━━━━
  [x] codebase_analyzer
  [x] task_formulator
  [ ] task_checker          ← running
  [ ] evaluation_checker
  [ ] eoh_runner
  [ ] results_checker
────────────────────────────────────────────────────────────
```

## Output Expectations

After execution, report:

- which mode was used,
- which agent steps were executed,
- where task files were written,
- where `runEoH.py` was written (self-contained, runs without external config),
- where run outputs were written,
- the best score and summary path when available.

---
> Source: [FeiLiu36/AutoEoH](https://github.com/FeiLiu36/AutoEoH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
