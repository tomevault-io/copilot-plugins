## cov-hunter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **First-read file:** `README.md` — project intent (coverage-hole → covIR → UVM case → run-local verdict), quick-start, and typical CLI pipeline. This file supplements the README; it does **not** repeat its content.
>
> Scope: this repo *is* the `cov_hunter` Python project (standalone, at `~/Projects/cov_hunter/`). It is an external CLI used against arbitrary UVM verification projects via `--project-root`. Keep the package self-contained — do not import from any target verification project, and do not have target projects import from `cov_hunter`.

## Install / test / run

Install editable into user site-packages — same convention as the sibling `take_root` project. Entry point lands at `~/.local/bin/cov-hunter` and is available from any cwd as long as `~/.local/bin` is on `$PATH`.

```bash
cd /home/robin/Projects/cov_hunter
python3.11 -m pip install -e .          # runtime only
python3.11 -m pip install -e .[dev]     # adds pytest for in-repo development

pytest                                   # all tests
pytest tests/test_parse_vcs_urg.py -k vcs_urg   # one file / one test
pytest -x --lf                           # stop on first fail, then rerun last-failed

cov-hunter <subcommand> --help           # installed as project.scripts entry point
python -m cov_hunter <subcommand> ...    # equivalent; useful when ~/.local/bin is not on PATH
```

The `cov-hunter` CLI also runs from a checkout without installing as long as `src/` is on `PYTHONPATH` (`PYTHONPATH=src python -m cov_hunter ...`).

Runtime deps are intentionally minimal: `PyYAML`, `jsonschema`. Dev only adds `pytest`. Python ≥ 3.10.

## Architecture — what the README's pipeline hides

The CLI verbs (`configure`, `doctor`, `init`, `parse`, `classify`, `review`, `report`, `case-gen`, `close-loop`, plus `run` as umbrella) are all thin wrappers. The actual layout is:

- **`cli.py`** — single argparse entry. Each subcommand's `set_defaults(handler=...)` points at a `phases/<name>.py` function. That's the only place new verbs get registered.
- **`phases/<name>.py`** — CLI glue only: resolve args, call into a domain module, write output. Keep business logic OUT of `phases/`.
- **Domain modules** (siblings of `cli.py`):
  - `covir.py` — the covIR dataclasses + JSON (de)serialization. covIR is the **single IR** that every later phase consumes; `schemas/covir.schema.json` is authoritative.
  - `adapters/__init__.py` — coverage adapter detection/dispatch. Current adapters are VCS `urg` HTML (`adapters/vcs_urg.py`) and Xcelium `xrun` IMC HTML (`adapters/xrun_imc.py`). New simulators/formats plug in as sibling adapters and must emit covIR; do not leak adapter-specific types past this boundary.
  - `scope_filter.py` — shared `coverage.dut_scopes` parsing and matching for adapters. Both VCS and XRUN accept `scope` for all descendants or `scope depth`, where depth `1` means current scope only, `2` means current plus one child level, and larger non-negative integers continue the same rule.
  - `classify_engine.py` — rule-based classification plus severity assignment driven by `profile.yml`. `severity_rules` only assign `S1-S4` to holes already marked `reachable`; they do not decide `reachable/deadcode/unr`.
  - `structural_unr.py` — conservative RTL/TB structural constant analysis used by classification. It indexes configured `rtl_roots` + `tb_roots`, propagates constant ties/assigns across module inputs, and fails closed on ambiguous or multiple drivers.
  - `review_engine.py` — optional LLM reviewer pass that annotates classified holes with `human_review` summaries, severity advice, and waive suggestions. It writes/reuses `review_shards/` when a run directory is provided.
  - `case_generation.py` — produces UVM test source and injects it into `test/uart_test_lib.sv` + `test/sv_list`, bounded by `marker_begin`/`marker_end` from the profile. Anything outside those markers is treated as human-owned and must not be touched.
  - `source_protection.py` — SHA-256 digest over RTL/TB files with AI-generated marker blocks stripped. `close_loop_engine.py` snapshots this digest before and after a rerun; mismatch ⇒ verdict fails (the loop is not allowed to mutate protected source).
  - `close_loop_engine.py` — invokes build/run/report commands under `build.build_dir` via `bash -lc` (typically after `setup_cmd`), captures per-case artifacts, and emits a **run-local** verdict. It intentionally does not merge or promote coverage — that's the outer project's job.
  - `doctor.py` — validates configured provider roles and optionally performs a minimal provider call, writing reports under `<project>/.cov_hunter/doctor/`.
  - `run_engine.py` — orchestrates `init → parse → classify → review → report → case-gen → close-loop` for one hole; used by `cov-hunter run`. `--report-only` stops after report, and `--skip-review` removes the review phase.
  - `reporting.py` — Markdown + JSON renderers from covIR.

- **Config layers** (three distinct files — don't conflate):
  1. `~/.cov_hunter/config.yaml` — global, provider + role mapping. Written by `configure`. Schema: `schemas/config.schema.json`.
  2. `<project>/cov_hunter.yml` — project manifest checked into the target verification project. This is the human-authored source of truth for hierarchy roots, coverage format/default HTML, DUT scope filters, test library paths, severity rules, case-generation conventions, and build commands. Schema: `schemas/project_manifest.schema.json`. Reference example: `profiles/uart_16550_example.yml`.
  3. `<project>/.cov_hunter/profile.yml` — normalized per-project runtime profile generated by `init` and refreshed by `run` unless an explicit existing `--profile` is supplied without `--manifest`. It expands the manifest with scanned `existing_tests` / `feature_to_test_map` data, stores `coverage_adapter`, and preserves normalized `dut_scopes`. Schema: `schemas/profile.schema.json`.
  4. `<project>/.cov_hunter/runs/<UTC-timestamp>/…` — per-run artifacts (`covir.json`, `covir.classified.json`, `covir.reviewed.json`, reports, shards, `case_gen/<hole_id>/`, logs, verdicts). `case-gen` / `close-loop` can infer the newest run when only `--hole` is provided.

- **`providers/`** — LLM backends behind a shared `base.Provider` interface: `claude_official`, `codex_official`, `anthropic_compatible` (covers Qwen/Kimi/DeepSeek via `base_url`). Roles (`classifier`, `case_writer`, `reviewer`) are chosen in `config.yaml` and their prompts live in `roles/*.md`. Default `codex_official` routing uses `gpt-5.5`; `configure` offers `gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, and `gpt-5.2`. To add a provider, implement `base.Provider` and register its `type` string — callers only reference providers by role, never by class.

## Conventions

- Every schema-backed artifact (config, profile, covIR) is validated with `jsonschema` on read **and** on write. When adding fields, update the JSON schema in `schemas/` first; tests pick it up automatically.
- The project-root path passed to most commands is the **target verification project** (an external path, e.g. `/path/to/uart_project`), not this `cov_hunter` repo. Both `cov_hunter.yml` and the generated `.cov_hunter/` tree live under *that* root.
- Tests under `tests/` use fixtures in `tests/fixtures/`. Snapshot-style tests (`test_report_snapshot.py`) compare against committed expected files — regenerate deliberately, not by blanket-updating.
- Adapter behavior should stay consistent across VCS and XRUN when it is controlled by common CLI/profile fields. In particular, keep DUT scope depth filtering in `scope_filter.py` and add paired adapter tests when changing it.
- Keep markers (`marker_begin`/`marker_end`) stable across releases; changing them invalidates existing injected cases and every stored source digest.

---
> Source: [gokeshenzhen/cov-hunter](https://github.com/gokeshenzhen/cov-hunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
