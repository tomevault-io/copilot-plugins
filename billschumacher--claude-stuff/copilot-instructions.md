## claude-stuff

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Two things in one repo:

1. **Claude Code plugin marketplace** — `.claude-plugin/marketplace.json` lists 17 plugins under `plugins/`. Each plugin contains skills as `plugins/<plugin>/skills/<skill>/SKILL.md` with YAML frontmatter. Owner: `BillSchumacher`.
2. **Evaluation framework** — Python 3.12 (Pipenv, stdlib only). Runs `claude -p` twice per test case (baseline vs with-skill), captures the full message stream, then scores via Opus 4.6 rubric judge, automated check scripts, and before/after diffs. Results are TSVs in `results/`. Currently 68 eval cases.

## Plugins

| Plugin | Purpose |
|---|---|
| `dev-workflow` | TDD with Gherkin, code quality, git discipline |
| `python-style` | Type annotations, docstrings, PEP 8, streaming |
| `secure-coding` | OWASP Top 10:2025, ASVS 5.0, NIST SSDF/CSF 2.0 |
| `api-design` | REST contracts: OpenAPI 3.1, RFC 9457, pagination, idempotency |
| `observability` | OpenTelemetry, structured logging, SLI/SLO |
| `acceptance-criteria` | Requirements specification: ISO 29148, BDD |
| `skill-orchestration` | Meta-skill for invoking ALL relevant skills, not just one |
| `efficient-code` | Language-neutral algorithmic efficiency (core) |
| `efficient-code-{c,cpp,csharp,go,javascript,php,python,rust,typescript}` | Language-specific efficiency: stdlib helpers + syntax/compiler/runtime gotchas |

## Commands

```bash
pipenv install --dev
pipenv run python -m src.cli list                         # list test cases (68)
pipenv run python -m src.cli run                          # run all (~2-3 hours)
pipenv run python -m src.cli run --cases "security_*"     # glob filter
pipenv run python -m src.cli report --run-id <ID>         # view a past run
pipenv run python -m src.cli new-plugin my-plugin \       # scaffold a plugin
  --description "One-line description"
pipenv run pytest tests/ -v                               # unit tests (18)
```

Full suite is 68 cases at ~2 min each. Use `run_in_background: true` when running from Claude Code.

## Plugin marketplace layout

```
.claude-plugin/
  marketplace.json                 # Catalog (name, owner, plugins[])
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json                  # name, description, version, author
    skills/
      <skill-name>/
        SKILL.md                   # YAML frontmatter + content
```

### Key behaviors

- Skills are auto-invoked by Claude based on the frontmatter `description`.
- **The agent only invokes ONE skill by default**, even when multiple match. Load `skill-orchestration` alongside worker skills for multi-skill tasks.
- Skills can influence how the agent WRITES new code but **cannot reliably make the agent REWRITE existing code** it wasn't asked to change. The system prompt's "don't change what you weren't asked to change" rule is stronger than skill directives.
- When given existing inefficient code to EXTEND, the skill sometimes overrides the existing pattern (Go `string +=` → `Builder`, Python list → set) and sometimes doesn't (C++ `auto` → `const auto&`, Python `sorted[:k]` → `heapq`). Patterns that look like "idiomatic correct code" are hardest to override.

### Adding a new plugin

Use the scaffold command — detects author from git remote:

```bash
pipenv run python -m src.cli new-plugin my-plugin \
  --description "One-line description" \
  [--skill my-skill]  # defaults to the plugin name
  [--author Name]     # override the detected author
```

Author detection (`src/git_meta.py`): parses `git remote get-url origin` owner segment (GitHub/GitLab/Bitbucket/SSH). Falls back to `git config user.name`, then `"unknown"`.

## Eval framework architecture

`src/cli.py` orchestrates the pipeline per case:

1. **`runner.py`** — builds the `claude -p` command. Uses `--output-format stream-json --verbose --dangerously-skip-permissions`. Subprocess calls use `encoding="utf-8", errors="replace"` (required on Windows — default cp1252 crashes on emojis/em-dashes). Baseline uses `--disable-slash-commands`; with-skill uses repeated `--plugin-dir <abs path>`. Each variant runs in an isolated temp dir. `run_claude()` returns `(text, messages)`. There is a separate `run_claude_json()` for `--output-format json` calls (scorer/differ use this).
2. **`scorer.py`** — Opus 4.6 judge call via `run_claude_json()`. Uses `--append-system-prompt` to instruct JSON output; parsed by `parse_judge_response()`. **Do not use `--json-schema`** — it caused API errors. `enrich_with_written_files()` appends `Write` tool contents so the judge sees actual code, not just the agent's summary.
3. **`checker.py`** — runs linters (per code block) and check scripts. Sets `PYTHONIOENCODING=utf-8` and `PYTHONUTF8=1` in child env. Passes `$EVAL_MESSAGES_FILE` (JSON path) and `$EVAL_EXPECTED_SKILLS` (comma-separated) to check scripts.
4. **`differ.py`** — `difflib.unified_diff` + Opus 4.6 diff summary.
5. **`results.py`** — streams rows to TSV incrementally.

`config.py` holds `RunResult(case_id, variant, raw_output, model, timestamp, messages)` and path constants.

## Test case TOML schema

```toml
[case]
id = "unique_id"
name = "Human-readable name"
description = "What the test measures"
plugins = ["plugin-name"]               # Plugins to load via --plugin-dir
expected_skills = ["plugin-name"]       # Optional; defaults to plugins.

[prompt]
text = """Neutral prompt that doesn't mention the skill area."""

[rubric]
criteria = ["Criterion 1", "Criterion 2"]   # Opus judge scores 0-2 each

[checks]
scripts = ["evals/checks/has_X.py"]
linters = ["ruff check"]

[options]
model = "sonnet"                        # Agent model (judge is always Opus)
max_budget_usd = 1.0
timeout_seconds = 600                   # Default 300
```

## Test case categories

Cases follow naming conventions:
- `efficiency_*` — write code from scratch, check for efficient patterns
- `refactor_*` — given inefficient existing code, extend it (tests if agent copies vs improves)
- `review_*` — given inefficient code, review and suggest fixes
- `security_*` — security-focused code generation
- `sdlc_*` — SDLC practices (API design, observability, acceptance criteria)
- `dev_workflow_*` / `combined_*` / `orchestrated_*` — workflow and multi-skill tests
- `python_style_*` / `streaming_*` — Python-specific style and streaming

## Check script conventions

- Live in `evals/checks/`. Files starting with `_` are shared helpers.
- Receive agent text on stdin. Read `$EVAL_MESSAGES_FILE` for full message stream, `$EVAL_EXPECTED_SKILLS` for expected skill list.
- Exit 0 = pass, non-zero = fail. Diagnostics to stderr.
- Import `_security_lib` for helpers: `get_all_code()` (Python, strips docs/comments), `get_all_code_c_style()` (C-family languages, strips `//` and `/* */`), `get_written_content()`, `fail()`.
- `get_all_code(languages=("python", "py"))` — set `languages` to match the fenced-code-block language tags you want.
- `get_all_code_c_style(languages=("go", "golang"))` — for JS/TS/Go/Rust/C/C++/C#/PHP.
- Stripping logic: `strip_docstrings_and_comments()` removes triple-quoted strings, non-f-string literals, and `#` comments. `strip_c_style_comments()` removes `//`, `/* */`, and string literals. This prevents false positives on patterns like `"never use verify=False"` in docstrings.

### Known check script issues

- **Python-centric checks produce false positives on other languages.** `no_sorted_for_minmax`, `no_naive_recursion`, and `no_double_lookup` use Python-specific regex. When attached to Go/JS/PHP cases, they can false-positive. Restrict these checks to Python-only cases, or write language-aware variants.
- **Review cases quote the original anti-pattern.** When the agent reviews code, it quotes the original (bad) code and then shows the fix. Checks that scan ALL code blocks will match the quoted original and fail. The C++ `no_range_for_copy_cpp` check hits this. For review cases, consider checking only the last code block or accepting "at least one block passes."

## Important gotchas

- **`--bare` flag has auth issues**: skips OAuth/keychain, only honors `ANTHROPIC_API_KEY`. Use `--disable-slash-commands` instead.
- **The judge sees only the agent's text response unless enriched.** `enrich_with_written_files()` in `scorer.py` fixes this by appending Write tool contents.
- **Each variant must run in its own working dir.** `_make_workdir()` in `runner.py` handles this. Without it, the with-skill variant inherits files from the baseline.
- **Windows path issues**: agent gets confused between `/tmp/` and `C:\Users\...\AppData\Local\Temp\`. Use relative paths or `tempfile.gettempdir()`.
- **Encoding**: `runner.py` and `checker.py` force `encoding="utf-8", errors="replace"` on all subprocess calls. `checker.py` also sets `PYTHONIOENCODING=utf-8` and `PYTHONUTF8=1` in child env. Without this, cp1252 on Windows crashes on emojis and em-dashes.
- **Open-ended prompts** cause baseline to dive into full implementation and time out. Either increase `timeout_seconds` or constrain the prompt.
- **Pyright "Import _security_lib could not be resolved"** on check scripts is expected — they use runtime `sys.path` manipulation. Ignore it.

## Conventions

- Functional style, no classes. Small focused functions.
- Memory-efficient: stream rows to TSV, don't accumulate.
- Result TSVs are gitignored except `results/.gitkeep`.
- All plugin authors detected from git remote (currently `BillSchumacher`).
- Judge model is Opus 4.6 (hardcoded default in `scorer.py` and `differ.py`). Agent model is set per case in TOML (default: sonnet).
- Efficiency eval cases target 4 per language (from-scratch, extend-existing, review, rule-coverage) across 9 languages: C, C++, C#, Go, JavaScript, PHP, Python, Rust, TypeScript.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BillSchumacher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
