## transcription-of-knowledge

> This repository contains a Python audio transcription pipeline.

# AGENTS.md

## Project Overview

This repository contains a Python audio transcription pipeline.

Main modes:

1. Local Whisper pipeline.
2. Remote GPU pipeline over SSH/SCP.

The project should stay simple, predictable, and safe for semi-automated edits.

## Main Goals

When working in this repo:

- keep changes minimal;
- avoid broad refactors;
- preserve existing behavior unless the task explicitly asks to change it;
- make the code easier to run, inspect, and maintain;
- make future Codex tasks safer and more deterministic.

## Progress Reporting

For non-trivial tasks, provide short progress updates during execution.

Progress updates are required when:

- starting a major phase of work;
- before running a command that may modify files, dependencies, environment, data, or generated outputs;
- before starting a potentially long-running process;
- after completing a meaningful milestone;
- when a blocker, uncertainty, failed command, or changed assumption appears.

Progress updates must be concise:

- one short message;
- no hidden reasoning;
- no step-by-step internal thought process;
- no repeated updates for small actions;
- no progress messages for trivial reads, searches, formatting, or tiny edits.

Use this format when practical:

`Progress: <phase> — <what is happening / what changed / what is blocked>`

For Python scripts:

- add progress indicators only when useful for long-running loops, file processing, API calls, batch jobs, or data pipelines;
- prefer `logging` or phase-based status messages for cron-ready, batch, pipeline, or production-like workflows;
- use `tqdm` only for interactive scripts or notebooks where a progress bar is useful and the dependency is already available;
- do not add a new dependency only for progress reporting unless explicitly requested;
- do not add progress indicators to tiny scripts, library functions, tests, or simple one-off helpers;
- progress reporting must not change financial logic, numeric calculations, deterministic outputs, schemas, or file locations.

When unsure, prefer minimal phase logging over a visual progress bar.

## Safety Rules

Do not modify these files or areas unless explicitly requested:

- `pipeline_config.json`
- `.env`
- `.env.*`
- browser cookies
- `output/`
- `downloads/`
- remote host settings
- SSH/SCP commands
- GPU/remote execution behavior

Do not delete, move, or clean generated files unless the task explicitly asks for cleanup.

Do not read secrets, cookies, local credentials, or private environment files.

## Source Selection Policy

The intended source selection rule is:

1. CLI `--url`
2. CLI `--input`
3. task file, if supported by the current code
4. config file values
5. built-in defaults

Important:

- CLI arguments should not be silently overridden by stale config values.
- `--url` and `--input` should not both be accepted silently.
- Any behavior change in source selection must be minimal and documented.

## Dependency Policy

Prefer explicit dependencies over runtime auto-installation.

When adding or updating dependencies:

- inspect actual imports first;
- add only confirmed non-stdlib dependencies;
- do not guess dependencies;
- do not pin versions unless there is a clear project reason;
- do not add unnecessary packages.

## Prompt Policy

Before changing prompt behavior:

1. Audit where prompts currently live.
2. Identify whether prompts are used locally, remotely, or inside generated helper scripts.
3. Do not change runtime prompt loading until the prompt flow is understood.
4. Do not change remote/GPU behavior without explicit approval.

Prompt files, when introduced, should be easy to edit and should clearly state:

- purpose;
- input;
- output;
- constraints;
- no invented facts.

## Validation

For code changes, run where practical:

```bash
python3 -m py_compile pipeline_common.py run_gpu_pipeline.py run_whisper_pipeline.py

---
> Source: [sergstack/transcription-of-knowledge](https://github.com/sergstack/transcription-of-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
