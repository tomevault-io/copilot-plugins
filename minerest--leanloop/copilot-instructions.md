## leanloop

> lean-loop is a delegation harness. **You (the frontier/cloud AI) are the

# CLAUDE.md — how an AI drives lean-loop

lean-loop is a delegation harness. **You (the frontier/cloud AI) are the
architect: you write the unit tests and the leanfile. The local model is the
laborer: it implements until the tests pass. Passing tests are the proof of
done — never the local model's prose.**

`leanloop.py` orchestrates: for each `[[tasks]]` entry it sends ONE fresh
prompt (task prompt + listed files + git diff, no chat history) to a wrapped
agent CLI backed by a local LLM, then runs the test command. Red tests enter
an auto-fix loop (up to `max_iters_per_task`); green tests advance to the
next task.

## The contract workflow

When asked to build a feature with lean-loop, follow this recipe:

1. **Design** the feature and identify the files involved.
2. **Write the unit tests yourself** — this is the contract. Be exhaustive:
   the local model will do exactly (and only) what the tests force it to do.
   Do NOT delegate contract-test writing to the local model.
3. **Write `leanfile.toml`** (schema below): one bite-sized task per slice
   of behavior, each scoped to its contract tests via `test_files`.
4. **Run** `./leanloop.py -c leanfile.toml` (or `lean-loop -c ...` if
   pipx-installed). The local LLM server must already be running at the
   health URL in `config.toml`.
5. **Verify yourself**: pass `--json-report report.json` and read the
   report (per-task status, fix iterations, error tails), then read
   `git diff`. Green contract = accept; anything else = tighten the tests
   or the task prompts and rerun (`--task <name>` reruns a single task).

## leanfile.toml — authoritative schema

Everything is optional unless marked REQUIRED. Paths are relative to
`defaults.project_root`. The file deep-merges on top of the global
`config.toml` shipped next to `leanloop.py` (which holds `[lean]` runtime,
`[health]`, and base `[defaults]` — don't repeat those per-project).

```toml
[runner]                       # the test command = the proof mechanism
command = "./venv/bin/pytest"  # REQUIRED. Any executable: pytest, go, npm...
args    = ["tests/", "-x", "--tb=native", "-q"]
timeout = 60                   # seconds, default 30 — raise for slow suites
# shell = true                 # run `command` as a shell line (ignores args)
# env   = { FOO = "bar" }      # extra env vars

[defaults]
project_root  = "."            # anchor for every relative path below
source_prefix = "src/"         # where source lives (string or list) — helps
                               # the pytest traceback parser target fixes
test_prefix   = ["tests/", "test_"]  # path fragments that mark test files
# max_iters_per_task = 5       # fix-loop attempts before the task FAILS
# summarize_errors   = true    # false = skip the diagnosis LLM call
# error_tail_lines   = 40      # error-output tail fed to the fix loop
# source_window      = 30      # lines of source around a failing line

[[tasks]]                      # PLURAL — [[task]] is silently ignored!
name   = "add-genre-filter"    # unique slug; used by --task
prompt = """
Add a `genre` query parameter to search_bands() in src/routes/search.py
that filters results by the Band.genre field. Do not modify any test files.
"""                            # REQUIRED — omitting it crashes the run
files  = ["src/routes/search.py", "src/models.py"]  # full contents sent as
                               # context; list ONLY what this task needs
test_files = ["tests/test_search.py", "-x", "-q"]   # REPLACES runner.args
                               # for this task (so re-add flags you need)
# context_budget = 24000       # context cap in CHARS (~4 chars/token)
# skip_tests = true            # scaffolding-only tasks; use sparingly
# after  = ["ruff check src/"] # shell commands run after the task passes
# runner = { timeout = 120 }   # per-task [runner] override (deep-merged)
protected = ["tests/test_search.py"]  # contract files: snapshotted before
                               # the task; if the model modifies them the
                               # task FAILS (even with green tests) and
                               # they are restored. Also valid under
                               # [defaults] to cover every task.
```

With zero `[[tasks]]`, lean-loop runs "direct mode": just the fix loop
against the current test failures. Useful for "make the suite green again".

## Writing tasks the local model can actually do

The local model is small. Each task gets ONE shot plus a few fix iterations,
and sees nothing but your prompt, the `files` contents, and the git diff.

- **One behavior per task.** "Add the endpoint" and "add validation" are two
  tasks. A feature is typically 3–8 tasks run in order.
- **Self-contained prompts.** Name the exact function, file, and expected
  behavior. Never reference "the previous task" or "as discussed" — there is
  no history.
- **Scope tests per task** with `test_files` so feedback is fast and
  failures unambiguous. Remember it replaces `runner.args` wholesale.
- **Protect the contract.** List your contract test files in `protected`
  (per task, or once under `[defaults]`). lean-loop snapshots them and
  fails any task that modifies them — passing by editing the tests is
  mechanically impossible, so a green run can be trusted without diffing
  the test files. Saying "do not modify test files" in the prompt still
  helps the model avoid wasting an iteration on the attempt.
- **Order matters.** Tasks run sequentially and later tasks see earlier
  edits via git diff. Scaffold → implement → integrate.

## Gotchas (silent failure modes)

- `[[task]]` (singular) parses fine and is **ignored** — lean-loop drops
  into direct mode with zero tasks. Always `[[tasks]]`.
- A task without `prompt` crashes mid-run (`KeyError`). `name` is optional
  but always set it.
- A wrong path in `files` does NOT abort — an inline `# ERROR` line is sent
  instead and the model works half-blind. Double-check paths against the
  real tree before running.
- `context_budget` is characters, not tokens; files past the budget get
  truncated silently.
- Unknown keys are ignored, not rejected. Typos (`test_file`, `promt`)
  vanish without a warning.

## CLI quick reference

```
./leanloop.py -c leanfile.toml        # run all tasks
./leanloop.py -c a.toml b.toml        # multiple configs, sequential
./leanloop.py -c leanfile.toml --task add-genre-filter
./leanloop.py -c leanfile.toml --tasks 2,3     # 1-based indices
./leanloop.py -c leanfile.toml --json-report report.json
                                      # machine-readable results: per task
                                      # status, fix iterations, error tail,
                                      # protected violations, token counts;
                                      # rewritten after every task — read
                                      # THIS after a run, not the log
./leanloop.py --init [FILE]           # emit an annotated sample leanfile
./leanloop.py --list-leaners          # show wrapper scripts (qwen, claude,
./leanloop.py --set-leaner aider.sh   #   aider, gemini, opencode)
```

## Repo layout (developing lean-loop itself)

- `leanloop.py` — the whole orchestrator. Python 3.11+, stdlib only; keep it
  dependency-free.
- `config.toml` — global static config (`[lean]` binary/model/base_url,
  `[health]`, base `[defaults]` incl. `global_message`).
- `leaners/` — thin wrapper scripts around agent CLIs; `[lean]` settings are
  exported as `LEAN_*` env vars before exec.
- `lean-mcp/server.py` — MCP server the *local* model uses (tools:
  `rg_search`, `rg_files`, `rg_context`, `run_tests`); requires `mcp` +
  ripgrep. `config.toml`'s `global_message` forces the local model onto
  these tools.
- `examples/` — leanfiles for pytest / go test / jest. `playground/demo/` —
  runnable end-to-end demo.
- Tests: `pytest tests/` (no venv tricks needed; stdlib project).

---
> Source: [Minerest/leanloop](https://github.com/Minerest/leanloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
