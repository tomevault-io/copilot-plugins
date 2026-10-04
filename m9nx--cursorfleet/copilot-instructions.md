## agents-md

> CursorFleet is a local TUI and workflow harness for Cursor agent teams. v0.1 is **Observe** only: no enforcement, no forced approvals, no cloud-agent visibility. Cursor is the only officially supported IDE; keep internals adapter-ready (`cursorfleet.adapters.*`, normalized events).

# CursorFleet: developer conventions

CursorFleet is a local TUI and workflow harness for Cursor agent teams. v0.1 is **Observe** only: no enforcement, no forced approvals, no cloud-agent visibility. Cursor is the only officially supported IDE; keep internals adapter-ready (`cursorfleet.adapters.*`, normalized events).

## Layout
- `src/cursorfleet/`: package (`cli`, `config`, `adapters/cursor`, `events`, `state`, `git`, `policy`, `workflow`, `tui`).
- `tests/{unit,integration,tui,security}`, `tests/fixtures/cursor` (sanitized real payloads only).
- `schemas/` JSON Schemas, `templates/cursor/` generated-kit templates, `docs/` documentation.
- Committed config: `.cursorfleet/`. Runtime state lives under `<git-common-dir>/cursorfleet/`, never in the working tree.

## Commands
- `uv sync` then `uv run ruff check .`, `uv run mypy src`, `uv run pytest`.

## Rules
- **Hook hot path is stdlib-only.** No pydantic/typer/textual imports (even transitively) in hook entrypoints; lazy-import everything else. Hooks must fail open: always exit 0 on any internal error, printing the right reply for the hook type: `{"permission":"allow"}` for permission hooks (`preToolUse`, `subagentStart`, `beforeShellExecution`, and `beforeMCPExecution`/`beforeReadFile` should they ever be wired), `{}` for all others (see `hook_policy.fail_open_response` and ADR 0004). An invalid reply from a permission hook blocks the action.
- **Every subprocess** uses an argv list (never `shell=True`) and an explicit `timeout=`.
- **Never persist** prompts, thinking/response text, file contents, command output, env vars, user emails or transcript paths. Drop them at the parser boundary using an allowlist. Never register `beforeSubmitPrompt`, `afterAgentThought`, `afterAgentResponse` or `beforeReadFile` hooks in v0.1.
- Paths stored are workspace-relative; anything outside workspace roots becomes `<external>`.
- **No sleeps in tests.** Use events, polling with deadlines on fakes, or Textual `run_test()` pilots.
- New CLI command groups go in their own module under `cli/commands/` exposing `register(app)`; do not edit `cli/main.py`.
- Rules in `.cursor/rules/` must be `.mdc` with frontmatter and under 500 lines.
- Python >=3.11; type-annotate everything (`mypy --strict`); keep Windows/macOS/Linux parity.

---
> Source: [M9nx/cursorfleet](https://github.com/M9nx/cursorfleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
