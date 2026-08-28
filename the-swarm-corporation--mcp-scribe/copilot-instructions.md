## mcp-scribe

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**mcp-scribe** turns an OpenAPI schema into a production-grade MCP server. It is a
compiler with a runtime attached: it lowers an OpenAPI document into MCP tools, then
executes those tools as real HTTP requests with auth, retries, rate limiting, and
response shaping already handled.

Package lives in `src/mcp_scribe/` (src layout, Poetry, `packages = [{include = "mcp_scribe", from = "src"}]`).

## Commands

```bash
poetry install --all-extras --with lint,test

poetry run pytest -q                        # 201 tests, ~1s
poetry run pytest tests/test_toolset.py -q  # one file
poetry run pytest -m remote                 # network tests (opt-in marker)

poetry run ruff check src tests             # lint
poetry run black src tests examples         # format  <-- see below
poetry run mypy src/mcp_scribe              # types
poetry build
```

### Formatting: black only. Do not run `ruff format`.

**black at `line-length = 70` is the formatter of record.** `[tool.black]` and
`[tool.ruff]` both pin 70 in `pyproject.toml`, and they must stay identical — when they
drift, each tool rewrites what the other just wrote and every file shows up as modified.

At 70 columns black and `ruff format` genuinely disagree (tuple-unpacking RHS,
`assert x, msg` parenthesization). They agreed at 100; they do not at 70. So:

- **Format with `black`.** Never run `ruff format` — it will fight black on ~2 files and
  reintroduce the churn loop.
- **Lint with `ruff check`.** That is ruff's only job here.
- `E501` is in ruff's `ignore` list. At 70 the remaining long lines are URLs, OpenAPI
  keyword names, and string literals no formatter can split; enforcing it produces 400+
  findings black cannot act on.

If a `ruff check --fix` result and black disagree, verify they reach a fixpoint (apply
the fix, run black, confirm nothing changes) before committing.

> `README.md`'s Development section still lists `ruff format --check src tests`. That
> command now fails by design — it should be `black --check`.

## Architecture

The pipeline is one directional flow. Each stage hands the next a narrower type.

```
spec/loader.py  ->  spec/parser.py  ->  toolset.py  ->  server.py  ->  runtime/executor.py
   fetch              normalize          lower          serve           execute
```

| Module | Responsibility |
| --- | --- |
| `spec/loader.py` | Fetch from URL / file / stdin. JSON or YAML. Caching, size limits. |
| `spec/refs.py` | Collect external `$ref` documents in one pass, load concurrently. |
| `spec/swagger2.py` | Convert Swagger 2.0 up to OpenAPI 3. |
| `spec/parser.py` | Dialect fixes, `allOf` flattening, lower to the internal IR. |
| `spec/model.py` | The IR: `ParsedSpec`, `Operation`. |
| `jsonschema.py` | Emit JSON Schema 2020-12, `$defs`, inlining, body flattening. |
| `naming.py` | `operationId` → tool name (snake case, length cap, collision handling). |
| `filters.py` | Tag / path / method / operation filtering. |
| `toolset.py` | `Toolset`, `ToolSpec` — the compiled output. |
| `server.py` | `build_server`, `load_toolset`, `ServerApp`, hot-swap refresh loop. |
| `runtime/executor.py` | One tool call → one HTTP request. Owns the pool. |
| `runtime/auth.py` | api_key / bearer / basic / oauth2 / static headers. |
| `runtime/passthrough.py` | Per-caller credential forwarding, allowlisted. |
| `runtime/resilience.py` | Retries, circuit breaker, token bucket. |
| `runtime/response.py` | Render an HTTP response into MCP content, truncate. |
| `runtime/serializer.py` | `style` / `explode` matrix, body encoding. |
| `config.py` | The whole `Settings` tree. Precedence, env mapping, `${VAR}`. |
| `cli.py` | Typer app: `serve`, `deploy`, `inspect`, `call`, `generate`, `install`, `version`. |
| `install.py` | Write MCP client configs (Claude Code/Desktop, Cursor, project). |
| `codegen.py` | `generate` — emit a standalone deployable project. |
| `banner.py` | Terminal presentation. Owns the brand violet and the ASCII mark. |

## Invariants worth preserving

These are load-bearing. Changing one means changing tests and probably the README.

1. **Nothing downstream of `spec/parser.py` touches a raw OpenAPI dict.** Spec quirks are
   handled once, at the boundary. If you find yourself reaching into a dict in
   `runtime/`, the fix belongs in the parser.

2. **Credentials never reach the model.** Parameters a configured credential supplies are
   stripped from the tool schema and injected at request time
   (`schema.hide_auth_parameters`). Do not weaken this.

3. **POST/PATCH are not retried by default.** `retry.retry_non_idempotent` is opt-in.
   Re-sending a billable request is worse than failing.

4. **Passthrough is an allowlist, and `authorization` is deliberately not in it.** Adding
   it silently would forward bearer tokens no one intended to forward.

5. **stdout is sacred on stdio.** JSON-RPC framing lives there. Banners, logs, and
   diagnostics all go to stderr. Colour is suppressed when not a TTY, when `NO_COLOR` is
   set, or when `TERM=dumb`.

6. **Config models are `extra="forbid"`.** An unknown key is a startup error, not a
   silent no-op. Keep it that way — it is the main defense against typo'd config.

7. **Settings precedence is defaults < config file < environment < flags.** Implemented
   in `Settings.load`; `None` overrides are pruned so unset flags don't clobber the file.

## Conventions

- **Line length 70**, black-formatted. See above.
- `from __future__ import annotations` at the top of every module.
- Pydantic v2 throughout. Secrets are `SecretStr`.
- Async everywhere in `runtime/` and `server.py`. `httpx.AsyncClient` with HTTP/2.
- Comments explain *why*, not *what*. The existing code is dense with rationale comments
  ("100 rather than the 70 used elsewhere: …"); match that register — a comment that
  restates the code is noise, one that records a decision is not.
- Errors live in `errors.py`, rooted at `OpenAPIMCPError`. `ReferenceError_` and
  `ValidationError_` carry trailing underscores to avoid shadowing builtins.
- The config key is `schema:`; the Python attribute is `Settings.json_schema` (`schema`
  is reserved on Pydantic models). Both validate, thanks to `populate_by_name`.

## Testing

- `pytest` with `asyncio_mode = "auto"` — async tests need no decorator.
- `filterwarnings = ["error::DeprecationWarning:mcp_scribe.*"]` — our own deprecation
  warnings are errors. Third-party ones are not.
- Fixtures in `tests/fixtures/`. `tests/conftest.py` holds shared setup.
- Tests marked `remote` hit a real API over the network and are opt-in.
- Per-file ignores let tests and examples import loosely (`F401`, `E402`, `E722`).

## Docs

- `README.md` — the landing page. Keep it tight.
- `docs/DOCS.md` — task-oriented guide.
- `docs/REFERENCE.md` — exhaustive CLI / config / env / Python API reference. Generated
  by reading the source, not by hand-tracking; if you add a config key or a flag, update
  the matching table there.
- `MCP_SCRIBE_SKILL.md` — how an agent should drive the CLI.

When you change a default in `config.py`, grep `docs/REFERENCE.md` for the old value.

## Gotchas

- **uvicorn starting is not a bug.** HTTP transport *is* uvicorn + Starlette. `serve`
  defaults to stdio and binds nothing; `deploy` is always HTTP.
- **`deploy` binds `0.0.0.0` but the banner prints `localhost`** (`cli.py`, `_print_banner`)
  — cosmetic, for clickability. The server really is on every interface.
- **HTTP transport needs the `http` extra.** Without `uvicorn`/`starlette` it raises a
  clear error rather than an ImportError traceback.
- `examples/` contains `petstore_readonly.py` and `swarms_api.yaml`. The Python example
  `swarms_api.py` is at the repo root, not in `examples/` — the README link to
  `examples/swarms_api.py` is broken.
- `.venv/` in this repo has `ruff 0.16.4`, but `pyproject.toml` pins `ruff <0.16.2` in the
  lint group. There is no lock file, so nothing breaks today, but the two disagree.

---
> Source: [The-Swarm-Corporation/mcp-scribe](https://github.com/The-Swarm-Corporation/mcp-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
