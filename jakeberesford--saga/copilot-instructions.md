## saga

> **Run `scripts/check.sh` before considering any change done.** It composes the

# AGENTS.md

## Validate every change

**Run `scripts/check.sh` before considering any change done.** It composes the
exact pieces CI runs — `scripts/lint.sh` (ruff check, ruff format --check, `ty`
type-check) then `scripts/test.sh` (pytest) — so the local gate, the release
gate, and CI never drift. A change is not finished until this passes. Treat a
failure as backpressure: fix it, don't work around it.

```sh
scripts/check.sh          # lint + type-check + test — the full gate
scripts/lint.sh           # lint + type-check only
scripts/test.sh           # tests only
```

Dev tools live in the PEP 735 `dev` group; install with `uv sync --group dev`.
Run a single test with `uv run pytest tests/test_model.py::test_name`. Auto-fix
formatting with `uv run ruff format .`.

### Mutation testing (optional, not part of the gate)

`scripts/mutation.sh` runs mutmut to check test *quality* rather than test
presence: it mutates the source and reports which mutations no test objects to.
Deliberately **not** in `scripts/check.sh` or CI. Install with
`uv sync --group mutation`.

`[tool.mutmut]` in `pyproject.toml` is the single source of truth for what gets
mutated (`only_mutate`) and what is suppressed. `uv run mutmut browse` shows each
survivor's diff interactively.

`scripts/mutation.sh` reports survivors in two groups, because mutmut's string
operator (which rewrites string *contents*: `"not found"` → `"XXnot foundXX"` or
`"NOT FOUND"`) accounts for roughly three quarters of them and would otherwise
bury the rest:

- **Structural** — the code around the strings changed. Triage these.
- **String contents only** — collapsed to a per-function count. Usually message
  wording, or a case flip of something case-insensitive anyway (HTTP header
  names and media types, Python codec names). Still listed, because a mutated
  string *can* be behavioral: a bogus header name or a rewritten allowed-origin
  literal lands here too.

The grouping is a reading order, not a suppression list. `uv run mutmut results`
still prints the flat list.

Triage every survivor into one of three buckets. The score is not a target —
driving it to 100% is impossible and the attempt produces brittle tests:

1. **A real gap** — the mutation is a behavior change nothing asserts. Write the
   test. This is the only bucket worth acting on.
2. **Unreachable code** — the mutation changes nothing because the code can't be
   reached. Delete the code and the mutant goes with it. Verify reachability
   first: a survivor is equally consistent with "dead code" and "untested
   load-bearing code", and those call for opposite fixes.
3. **A provably equivalent mutant** — identical observable behavior, so no test
   can kill it. Leave it. Every suite has some; they are a floor, not a backlog.

Message prose is unkillable by design: tests assert on message *substrings*
(`match="no chapters"`) precisely so wording stays free to change. Don't pin
message text to kill a mutant.

`do_not_mutate_patterns` suppresses prose that sits on a line of its own. It
cannot help with `self._send_json(404, {"error": "not found"})`, because mutmut's
suppression — patterns and `# pragma: no mutate` alike — is **line-granular**,
and that line also carries the status code and the `error` key. Both are
load-bearing (real gaps have been found in both), so silencing the line to
silence its prose would cost more than it saves. That is why the split happens at
report time instead. Audit any new pattern against the source before trusting it:
the original prose regex silently suppressed `serve.py`'s allowed-origin literals,
which are behavioral.

## What saga is

`saga` is a CLI that turns a git diff into one self-contained static HTML page:
a chapter-by-chapter guided tour of a change. It partitions the whole diff into
ordered chapters via a single structured LLM call, then inlines everything
(diff2html, marked, the data) into one file that opens offline.

## Architecture

The pipeline runs left to right; `cli.py` (`main`) orchestrates it:

**diff → generate → render → (serve → comments)**

- **`diff.py`** — computes the change set. `compute_diff` diffs two refs purely
  from git (`base...ref`, no checkout); `pr_diff` fetches the same shape from a
  GitHub PR via the `gh` CLI. Both produce a `DiffResult`. `default_base`
  auto-detects the base ref. Stdlib + the `git`/`gh` CLIs only.

- **`model.py`** — the pure, stdlib-only core. `parse_hunks` splits a unified
  diff into stable-id (`h0, h1, …`) `Hunk`s; `reconstruct_diff` rebuilds a valid
  unified diff for any subset so the browser renders one chapter with the same
  anchors as the full view. `Chapter`/`Saga` are the persisted dataclasses.
  `validate_coverage` enforces **the hard invariant: every hunk belongs to at
  least one chapter** — it raises `SagaError` on any gap, and nothing the LLM
  returns is trusted until it passes.

- **`generate.py`** — one structured LLM call. The prompt (`prompts/saga.md`,
  shipped in the wheel) plus the labeled diff go in; schema-validated JSON comes
  back. Provider is a `provider/model` string dispatched through `instructor`
  (`anthropic`/`openai`/`openrouter`, `local/` for Ollama/LM Studio, and
  `claude-cli` which shells out to the logged-in Claude Code CLI). A chapter-count
  cap (`_MAX_CHAPTERS`) is a pydantic validator so an over-limit saga is retried
  smaller rather than discarded. `verdict()` counts are recomputed in code, never
  trusted from the model.

- **`render.py`** — builds the single HTML file. Inlines design tokens,
  vendored CDN bundles (cached under `~/.cache/saga`), the saga JS, and the
  payload. Assets live in `assets/`. `file_links` tells the client how to turn
  diff paths into links (local editor URL vs. GitHub blob URL). The page also
  gets an empty comments block minted with a fresh `sagaId`.

- **`serve.py`** — `saga serve`, a loopback-only HTTP writer. The browser can't
  write its own file portably, so this server does. It holds **no state**: the
  HTML file is the durable store. Security matters even locally — binds
  `127.0.0.1`, validates Host/Origin (anti-DNS-rebind), no CORS, per-run
  in-memory write token. Port is derived from the `sagaId` so the origin (hence
  `localStorage`) is stable across restarts.

- **`comments.py` / `comments_block.py`** — review comments live as a JSON
  envelope spliced between sentinel comments **inside** the saga HTML.
  `comments_block.py` is the only code that reads/rewrites that block (atomic,
  byte-preserving splice). `comments.py` consumes it: `push` posts a single
  PENDING GitHub review via `gh`; `read` emits normalized JSON for a coding
  agent. Comments only ever come from the file — there is no hand-authored input.

The front-end (`assets/saga.js`, `saga-merge.js`, CSS) is inlined verbatim; it
reads `window.__sagaData` and talks to `serve.py`'s `/api/*` routes.

## Conventions

- **Errors:** raise `SagaError` (from `model.py`) for anything user-facing; the
  CLI catches it and prints `error: <msg>` to stderr with exit 1.
- **Tests** use real throwaway git repos (`git_repo` fixture) — git is never
  mocked. The one LLM HTTP call is replayed from a VCR cassette
  (`tests/cassettes/`, auth scrubbed); tests never hit the network. Re-record
  with `SAGA_RECORD=1` and a real key.
- **Package data:** `prompts/` and `assets/` ship inside the wheel and resolve
  relative to the package at runtime — reference them via
  `Path(__file__).resolve().parent`.

---
> Source: [JakeBeresford/saga](https://github.com/JakeBeresford/saga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
