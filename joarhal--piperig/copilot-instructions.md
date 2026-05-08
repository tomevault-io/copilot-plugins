## piperig

> Declarative pipeline runner. Go. Single binary.

# piperig

Declarative pipeline runner. Go. Single binary.

## Read order
1. `docs/SPEC.md` — format, API, examples. **Start here.**
2. `docs/ARCHITECTURE.md` — packages, data flow, dependency graph.
3. `docs/TESTING.md` — test strategy, cases, fixtures.
4. This file — development rules.

## Architecture

```
embed.go                          go:embed README.md for `piperig llm`
cmd/piperig/main.go               CLI entrypoint
internal/
  pipe/                            types (Pipe, Step, Call, Plan) + YAML parsing
  timeexpr/                        time expressions (-1d, -2h, -1w, ranges)
  expand/                          loop/each/template expansion → Plan
  validate/                        12 validation rules, before execution
  runner/                          subprocess execution, stdout JSON parsing
  output/                          formatted terminal output (icons, colors)
  scheduler/                       cron + every, schedule YAML
  picker/                          TUI interactive mode (bubbletea)
  config/                          .piperig.yaml (interpreter overrides)
```

## Rules

1. **English only.** All code, comments, commit messages, docs, and issues in English. Public project.
2. **SPEC.md is the source of truth.** Implementation follows the spec, not the other way around. If something in code conflicts with the spec — fix the code.
3. **Minimal external dependencies.** stdlib + `gopkg.in/yaml.v3` + `bubbletea` for TUI + cron library for scheduler. Nothing else.
4. **Test with `go test`.** Every package has `_test.go`. Expansion is pure logic — test it thoroughly with table-driven tests. Run `go test ./... -count=1` before committing.
5. **testdata/ contains example pipes.** Tests load from there. Keep examples in sync with SPEC.md.
6a. **Every feature must have an E2E test.** If you add a feature to the spec — add an E2E test that covers the full path: YAML → binary → stdout/exit code. No exceptions.
6. **Process boundary.** piperig calls a subprocess, passes params via env/json/args. Exit code determines success (0) or failure (!=0). No Go plugins, no shared memory, no RPC.
7. **Steps are sequential.** No parallelism within a pipe. One step finishes, next begins.
8. **Fail fast.** Non-zero exit code stops the pipe (unless `allow_failure: true` on step).
9. **Pipe files use `.pipe.yaml` extension.** This is how piperig identifies pipes among other YAML files.
10. **Job is a file path.** Resolved relative to cwd. Execution method determined by extension (`.py` → python, `.sh` → bash, no extension → direct exec).
11. **`with` not `props`.** Parameters passed to jobs are called `with` in pipe YAML.
12. **Call is the central type.** Everything either produces Calls (expand) or consumes them (runner, check).

## Commits

1. **Atomic.** One logical change per commit. Don't mix refactoring with new features.
2. **Conventional Commits format.** `type(scope): description` — types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`. Scope is the package name (`pipe`, `timeexpr`, `expand`, etc.) or `cli` for cmd/.
3. **Imperative mood.** "add parser" not "added parser".
4. **Body when needed.** If the change isn't obvious from the title, add a blank line and a short body explaining _why_.
5. **Co-author.** Every commit includes `Co-Authored-By` for all contributors.
6. **English only.** Commit messages in English.
7. **No version tag without docs.** Do not tag a release until all documentation (SPEC.md, ARCHITECTURE.md, TESTING.md, README.md) is updated for the change. Docs ship in the same commit or before the tag — never after.

## Screenshots

Terminal screenshots for README and docs are generated from HTML templates.

```
assets/
  terminals/
    terminal.css        shared styles (colors, fonts, layout)
    banner.html         each screenshot is a separate HTML file
  generate.sh           converts all HTML → PNG (uses Puppeteer)
```

- Each HTML file links `terminal.css` and contains only the terminal content.
- CSS classes match piperig output: `.cmd`, `.job`, `.timestamp`, `.dim`, `.ok`, `.fail`, `.warn`, `.retry`, `.summary-ok`.
- PNGs are generated with transparent background and saved to `assets/`.
- Run `make screenshots` to regenerate all PNGs.
- To add a new screenshot: create `assets/terminals/<name>.html`, run `make screenshots`.

## Testing

Three levels of tests, all run with `go test ./...`:

1. **Unit tests** — pure logic in each package (`expand/`, `validate/`, `timeexpr/`, `pipe/`, `output/`). Table-driven, deterministic, no I/O.
2. **Integration tests** — `runner/` tests with real shell scripts from `internal/runner/testdata/scripts/`. Test subprocess execution, retry, timeout, signal handling.
3. **E2E tests** — `cmd/piperig/` builds the binary once in `TestMain`, then each test writes temp files and runs the binary as a subprocess. Tests the full path: YAML → CLI → stdout/exit code.

### E2E test structure

E2E tests are split by domain — one file per area:

```
cmd/piperig/
  main_test.go              helpers (run, writeFile, writeScript) + TestMain + basic CLI tests
  e2e_multistep_test.go     multi-step sequential execution, fail-fast
  e2e_iteration_test.go     loop (numeric, list, time range, cartesian), each, each+loop
  e2e_execution_test.go     retry, timeout, allow_failure
  e2e_input_test.go         input modes (env, json, args), direct exec
  e2e_nested_test.go        nested pipes, parent override
  e2e_template_test.go      template substitution from loop/each values
  e2e_output_test.go        JSON log fields, stderr, summary, pipe header
  e2e_validation_test.go    all validation error cases (exit code 2)
  e2e_check_test.go         check command (directory, overrides, description)
  e2e_config_test.go        .piperig.yaml (custom interpreters, process env)
  e2e_schedule_test.go      serve --now, schedule overrides
  e2e_scaffold_test.go      init/new edge cases (already exists)
  e2e_dotenv_test.go        .env loading, priority, interpolation
  e2e_hooks_test.go         on_fail/on_success hooks (env vars, stdin, override, false)
```

### Rules for writing tests

- **One feature per test.** Name: `TestArea_Scenario` (e.g. `TestLoop_NumericRange`).
- **Assert exit code first**, then check stdout/stderr content.
- **Use `check` for counting calls** — fast, no execution. Assert on `"Total: N calls"`.
- **Use `run` for verifying output** — actual execution with real scripts.
- **New feature = new E2E test.** Every capability from SPEC.md must have a corresponding E2E test. If a feature is in the spec but missing a test — that's a bug.

# ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in docs/plans/PLANS.md) from design to implementation.

---
> Source: [joarhal/piperig](https://github.com/joarhal/piperig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
