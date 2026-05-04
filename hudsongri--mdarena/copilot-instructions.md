## mdarena

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                                          # Install/update dependencies
uv run pytest                                    # Run all tests
uv run pytest tests/test_file.py::test_name -v   # Run a single test
uv run ruff check src/ tests/                    # Lint
uv run ruff format src/ tests/                   # Format
uv run ty check src/                             # Type check
```

## Architecture

Three-stage pipeline: **mine** (fetch PRs, build task set) → **run** (agent A/B test) → **report** (score and compare).

### Mining (`mining/`)
- `pr_fetcher.py` fetches merged PRs via `gh` CLI, `pr_filter.py` excludes bots/lockfiles/size outliers, `task_builder.py` builds `TaskInstance` objects.
- With `--detect-tests`, test extraction runs the test suite at `base_commit` and `merge_commit` to compute FAIL_TO_PASS / PASS_TO_PASS lists. This uses the same repo cache (`~/.mdarena/repos/`) but checks out different commits.

### Runner (`runner/`)
- Each task x condition pair gets an isolated **git worktree** (not Docker).
- `sandbox.py` creates the worktree, strips ALL CLAUDE.md/AGENTS.md from the entire tree via `rglob`, then injects the condition's context file(s). Accepts a single file (placed at root) or a directory (recursively copied, preserving paths for monorepo support).
- `agent.py` invokes Claude Code by piping the prompt via **stdin** (interactive mode with tool use), not `--print`. Baseline uses `--bare`; context conditions omit `--bare` so Claude auto-discovers the injected CLAUDE.md. This matches how Claude Code is evaluated on SWE-bench.
- Patch is extracted via `git add -N . && git diff HEAD --no-color --no-ext-diff`.

### Evaluation (`evaluation/`)
- **Dual scoring:** test pass/fail (binary, same as SWE-bench) when FAIL_TO_PASS/PASS_TO_PASS exist, diff overlap (Jaccard of files and line numbers) otherwise.
- `metrics.py` blends both: test result where available, file overlap > 0.5 threshold where not. Report shows both metrics.
- Statistical significance via paired t-test (requires >= 5 shared tasks).

### Discovery (`discovery/`)
- Auto-detects test commands from CI/CD files and package manifests. Three tiers: GitHub Actions YAML → package files (package.json, pyproject.toml, Cargo.toml, go.mod) → user override via `--test-cmd`.
- `output_parsers.py` parses test runner output (pytest, jest, go test, cargo test, JUnit XML) to extract individual test names and pass/fail status.

### SWE-bench Compatibility
- `TaskInstance` model carries both mdarena-specific fields (pr_number, files_changed) and SWE-bench fields (FAIL_TO_PASS, PASS_TO_PASS, version). All SWE-bench fields are optional with defaults.
- `load-swebench` imports from HuggingFace datasets; `export-swebench` outputs JSONL. Predictions are auto-written in SWE-bench format after each run.

## Key Design Decisions

- All subprocess wrappers (`utils/git.py`, `utils/gh.py`) have explicit timeouts (120s for gh, 300s for git, 600s for clones). Never add a subprocess call without a timeout.
- All `except` clauses catch specific exceptions, no bare `except Exception:`.
- `RunCondition.context_file` can be a file path or directory path. The sandbox handles both.
- Cost/usage parsing from Claude CLI output is heuristic and may return 0.0 if the format changes.

## Commit Style

Plain English summaries, no conventional commit prefixes. Describe what changed and why.

---
> Source: [HudsonGri/mdarena](https://github.com/HudsonGri/mdarena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
