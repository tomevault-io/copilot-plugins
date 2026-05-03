## pike-agent

> pike-agent is an AI agent that analyzes strace logs. It indexes strace output into a SQLite database (syscalls table + per-argument rows with type tags), then exposes query tools to an LLM so it can answer questions like "what does this program do?", "is it malware?", "which network peers does it communicate with?".

# AGENTS.md

## Project

pike-agent is an AI agent that analyzes strace logs. It indexes strace output into a SQLite database (syscalls table + per-argument rows with type tags), then exposes query tools to an LLM so it can answer questions like "what does this program do?", "is it malware?", "which network peers does it communicate with?".
The LLM interprets raw strace argument syntax directly — no syscall-specific semantic extraction. Supports local llama.cpp inference and external APIs (Claude, Gemini, etc.).

## Architecture

- **Parser** (`pike_agent.strace.parser`): Parses strace lines into typed `Syscall` records via a regex envelope and top-level argument splitting, with `parse_log()` merging unfinished/resumed pairs across interleaved processes.
- **Pipeline Orchestrator** (`pike_agent.strace` + `pike_agent.strace.run`): Runs `strace` into a FIFO and executes a bounded threaded pipeline (`read_pipe` → `parse_log` → DB writer), with unified `StraceError` handling.
- **Storage Layer** (`pike_agent.db`): Uses `WriteDb` for batch inserts and `finalize()` (indexes + FTS5), then exposes `ReadDb` for read-only SQL with a row-limit safeguard.
- **CLI + Configuration** (`pike_agent.cli`, `pike_agent.config`): Exposes `run`/`attach`/`chat`, configures logging, and loads TOML model config from XDG paths with default local setup and keyring API key lookup.
- **Agent Runtime** (`pike_agent.agent`): Builds a pydantic-ai agent with model/provider setup, wires tools and system prompt, and orchestrates the chat entry point with Rich-based logging. Sub-modules: `deps` (`AgentDeps` dataclass carrying `ReadDb`), `tools` (DB query + man-page lookup tools), `prompt` (system prompt template + live schema), `chat_ui` (streaming terminal chat loop with thought blocks, tool transcripts, and Rich Live rendering).
- **Man Page Utilities** (`pike_agent.man`): `man 2` helpers for full-page reads, section extraction, and keyword-context search.

## Code Style

- Python 3.13+
- Dataclasses for all structured data.
- No `_` prefix on any name (methods, functions, variables, attributes) unless it is genuinely unused (e.g. `for _ in range(n)`). All names are plain, even internal helpers.
- Do not use `del` to discard unused function parameters. Instead, prefix the parameter name with `_` in the signature (e.g. `_prompt: str`).
- Docstrings mandatory on all functions (imperative mood).
- Typing:
  - Annotations mandatory on all function signatures. Always write the real type, never a string-quoted annotation.
  - Use `from __future__ import annotations` only when genuinely required for unresolved forward references.
  - Avoid `typing.Any`; use precise types, protocols, or generics instead. `Any` is acceptable only as a last resort when no precise type is feasible, never as a shortcut to skip proper typing.
  - Avoid `typing.cast`; prefer precise annotations, runtime narrowing (`isinstance` / assertions), or API shapes that type-check without casts.
- No verbose comments that paraphrase the code.
- Split large functions into small, single-responsibility ones when needed.
- Favor importing the root module and using fully qualified names in code. Exceptions: names from `typing`, and `pathlib.Path`.
- **IMPORTANT: Never inline raw escape codes, magic strings, thresholds, or unexplained literal values. All such values must be defined as named constants (module-level or class-level). No exceptions.**
- Never use `""` or `0` as sentinel values to mean "absent" or "not set". Use `None` (with `| None` in the type annotation) so the type system distinguishes missing from legitimately empty/zero.
- Group all module-level constants at the top of the file, before class and function definitions.
- Do not add large section-separator comment blocks (e.g. `# ===...` banners). Use class docstrings and natural whitespace to organize code.
- At the end of any refactor, remove dead code (unused constants, types, helpers, and imports) before finishing.
- Always favor f-strings for string formatting, unless there is a specific reason not to (e.g., logging lazy formatting with `%s` is acceptable when performance matters).
- All logging messages must start with a capital letter.

## Linting & Formatting

Code must pass all three:

```sh
uv run ty check
ruff check
ruff format --check
```

## Bug Fixes

Unless stated otherwise, always use a red-green testing approach:

1. Investigate the bug and write a failing test that exercises the code path.
2. Implement the proper fix.
3. Check the added tests now pass successfully.

## Testing

- All style/format/lint guidelines apply to main code **and** test code.
- Tests use `unittest` (stdlib only).
- Do not pass a `msg` argument to `assert*` calls. Let the assertion's default failure render the values; if extra context is needed, refactor the surrounding code (e.g. helper, context manager) instead of stuffing it into a string message.
- Run all tests: `uv run python -m unittest discover -s tests`
- Run a single test module: `uv run python -m unittest tests.test_strace`
- Run a single test: `uv run python -m unittest tests.test_strace.TestParseLines.test_parses_into_batches`
- `tests.test_llm_eval` contains optional live LLM evals over frozen strace logs. They are skipped by default unless `PIKE_TEST_MODEL` is set.
- Example with a local llama.cpp OpenAI-compatible server:

```sh
PIKE_TEST_MODEL=openai:default \
PIKE_TEST_BASE_URL=http://localhost:8000/v1 \
uv run python -m unittest tests.test_llm_eval
```

- If the provider requires it, also set `PIKE_TEST_API_KEY`.

## Benchmarks

Pipeline benchmarks live in `benchmarks/bench_pipeline.py`. They cover parsing, DB insert, finalize (index + FTS), and end-to-end. Set `BENCH_LOG` to an strace log file and run:

```sh
BENCH_LOG=gimp.log uv run python benchmarks/bench_pipeline.py --repeat 3
```

## Version control

- This repository uses the jujutsu VCS. **Never use any `jj` command that modifies the repository**.
- You can also use read-only git commands for inspecting repository state. **Never use any git command that modifies the repository**.

---
> Source: [synacktiv/pike-agent](https://github.com/synacktiv/pike-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
