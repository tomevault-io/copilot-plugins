## logseq-cli

> This repository currently contains a Python CLI for the Logseq local HTTP API.

# logseq-cli

## Scope

This repository currently contains a Python CLI for the Logseq local HTTP API.

This file is intentionally derived from the codebase, not from project docs.

## Source Of Truth

When deciding what is supported, use this order:

1. `tests/`
2. `src/`
3. `pyproject.toml`

If documentation disagrees with code or tests, trust the code and tests.

## Unsupported Method Tracking

Maintain `UNSUPPORTED-LOGSEQ-HTTP-METHODS.md` as the ledger of live-tested
Logseq HTTP methods that turned out to be unsupported.

When a method fails against a real running Logseq server because it does not
exist or is unsupported:

- add an entry to `UNSUPPORTED-LOGSEQ-HTTP-METHODS.md`
- include the exact method name
- include the date discovered
- include the exact command or reproduction steps
- include the returned server error
- update the CLI, tests, and docs so the repo stops claiming that method works

Do not leave disproven HTTP methods documented as supported.

## Stack

- Python package with entry point `logseq = "src.cli.main:app"`
- CLI framework: `typer`
- HTTP client: `httpx.AsyncClient`
- Test stack: `pytest`, `pytest-asyncio`, `typer.testing`

## Runtime Model

- The CLI talks to Logseq over HTTP.
- `LogseqClient` posts JSON requests to `http://127.0.0.1:12315/api` by default.
- Requests use the shape `{ "method": ..., "args": [...] }`.
- Auth is a bearer token resolved from `LOGSEQ_TOKEN` or the stored CLI auth token.
- The CLI-managed token setup commands live under `logseq auth`.
- Missing auth configuration must fail clearly with a non-zero exit.
- In installation and setup docs, treat configuring auth as a required installation step, not optional post-install configuration.

Relevant files:

- `src/logseq_client.py`
- `src/logseq_service.py`
- `src/cli/main.py`

## Implemented Command Groups

Top-level groups registered in `src/cli/main.py`:

- `version`
- `auth`
- `page`
- `block`
- `graph`
- `query`

### `auth` commands

Implemented in `src/cli/auth.py`:

- `set-token`
- `status`

### `page` commands

Implemented in `src/cli/page.py`:

- `list`
- `get`
- `create`
- `delete`
- `rename`
- `refs`
- `properties`
- `journal`
- `ns-list`
- `ns-tree`

### `block` commands

Implemented in `src/cli/block.py`:

- `get`
- `insert`
- `update`
- `remove`
- `prepend`
- `append`
- `move`
- `collapse`
- `properties`
- `prop-set`
- `prop-remove`
- `insert-batch`

### `graph` commands

Implemented in `src/cli/graph.py`:

- `info`

### `query` commands

Implemented in `src/cli/query.py`:

- `run`

## Output Contract

Keep this stable unless there is a strong reason to change it.

- Default stdout format is NDJSON.
- A list result prints one JSON object or JSON array row per line.
- A single object prints as one JSON line.
- `--plain` switches to human-readable `key: value` output.
- `--fields` filters keys when the command supports it.
- stdout should stay clean for piping.
- Errors belong on stderr.

Relevant files:

- `src/cli/output.py`
- `tests/test_output.py`
- `tests/test_errors.py`

## Stdin Contract

Several commands support piped NDJSON when a positional identifier is omitted.

Implemented behavior:

- `read_stdin_field("name")` extracts `.name` values from NDJSON lines
- `read_stdin_field("uuid")` extracts `.uuid` values from NDJSON lines
- TTY stdin returns an empty list
- Missing required fields in piped NDJSON raise a clear error

Commands relying on stdin-driven identifiers include:

- `page get`
- `page delete`
- `block get`
- `block insert`
- `block remove`

Relevant files:

- `src/cli/stdin.py`
- `tests/test_stdin.py`

## Error Handling Rules

Preserve the current UX:

- Connection failures should print `Cannot connect to Logseq` on stderr and exit 1.
- HTTP status failures should include the response status code on stderr and exit 1.
- Missing required CLI args should exit non-zero.
- Token/config errors should fail early and clearly.

Relevant coverage:

- `tests/test_errors.py`

## Service Layer Expectations

`src/logseq_service.py` is the API-mapping layer. Prefer adding new Logseq-facing
operations there before wiring them into the CLI.

Current service responsibilities include:

- page retrieval and mutation
- block retrieval and mutation
- graph info
- query execution
- namespace listing/tree retrieval
- property operations

`page properties` is implemented as a fallback over the first block returned by
`logseq.Editor.getPageBlocksTree`, because the raw
`logseq.Editor.getPageProperties` HTTP method is unsupported on the tested live
server.

`page journal` is implemented as a fallback that calls `createPage` with a
`YYYY_MM_DD` name, then resolves the resulting journal page via its
`journalDay`, because the raw `logseq.Editor.createJournalPage` HTTP method is
unsupported on the tested live server.

`block append` is implemented as a fallback that reads the page tree and inserts
after the last top-level block as a sibling, or prepends when the page is
empty, because the raw `logseq.Editor.appendBlock` HTTP method is unsupported on
the tested live server.

Keep the CLI thin and the service layer explicit.

## Change Rules For Codex

- Do not add undocumented claims about unsupported subsystems.
- Do not leave disproven HTTP methods exposed in the CLI.
- Do not remove NDJSON-by-default behavior.
- Do not break piping behavior casually.
- Do not move error text from stderr to stdout.
- When changing a command contract, update tests in the same task.
- When adding a new command or changing an existing command, update the README in the same task.
- When adding a new command or changing an existing command, verify it is represented in the CLI help output in the same task.
- When making a code or user-facing behavior change in `src/` or the packaged CLI, also increment the version in `pyproject.toml`.
- Do not bump the version for documentation-only, comment-only, or other non-code changes.
- In docs and examples, do not present any installation flow as complete unless it also includes the required auth setup step.
- Follow semantic versioning for version bumps unless the user explicitly asks for a different scheme:
  - patch (`0.1.1`) for bug fixes and small backward-compatible internal improvements
  - minor (`0.2.0`) for new backward-compatible features
  - major (`1.0.0`) for breaking changes
- Prefer targeted edits over broad rewrites.
- Keep command naming consistent with the existing noun-verb structure.

## Preferred Validation

When changing behavior, use targeted tests first, then broader tests as needed.

Typical commands:

```powershell
pytest --tb=short
pytest tests/test_page.py
pytest tests/test_block.py
pytest tests/test_query.py
pytest tests/test_graph.py
pytest tests/test_errors.py
pytest tests/test_output.py
pytest tests/test_stdin.py
```

## Practical Editing Checklist

Before finishing a change, verify:

- Is the feature actually implemented in `src/`?
- Is the behavior covered by tests?
- Is stdout still pipe-safe?
- Are errors still on stderr?
- Did stdin-based workflows keep working?
- Did you avoid introducing claims about unsupported features?
- Did you update `UNSUPPORTED-LOGSEQ-HTTP-METHODS.md` if a live test disproved a method?

---
> Source: [wolf-jonathan/logseq-cli](https://github.com/wolf-jonathan/logseq-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
