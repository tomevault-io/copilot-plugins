## clawlite

> - Repo: https://github.com/renan/ClawLite (update with real URL)

# AGENTS.md — ClawLite Repo Guide

- Repo: https://github.com/renan/ClawLite (update with real URL)
- In chat replies, file references must be repo-root relative only (example: `clawlite/channels/telegram.py:80`); never absolute paths.
- GitHub issues/comments/PR bodies: use literal multiline strings or `-F - <<'EOF'` for real newlines; never embed `\n` as text.
- GitHub comment footgun: never use `gh issue/pr comment -b "..."` when body contains backticks or shell chars. Always use `-F - <<'EOF'` heredoc.
- When working on a GitHub Issue or PR, print the full URL at the end of the task.
- When answering questions, respond with high-confidence answers only: verify in code; do not guess.
It captures the repo's actual commands, validation workflow, and coding style.

## Priority
1. Follow safety and applicable law.
2. Follow the user's explicit request.
3. Follow repository-specific guidance in this file.
4. Prefer small, auditable changes over broad refactors.

Core behavior inherited from the previous local guide:
- Be objective, technical, and useful.
- Ask questions only when critical information is missing.
- Prefer execution plus verifiable results.
- Validate with tests or smoke checks when practical.
- Explain change impact clearly.
- Never expose secrets or use destructive commands casually.

## Rule Files Found
- Root guidance exists in this `AGENTS.md`.
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.

## Repo Snapshot
- Language: Python.
- Packaging: `pyproject.toml` with `setuptools`.
- Python: `>=3.10`.
- Package: `clawlite`.
- CLI entrypoint: `clawlite = clawlite.cli:main`.
- Gateway: FastAPI + uvicorn + WebSocket support.
- Runtime autonomy already includes supervised background loops for `heartbeat`, `autonomy`, `channels_dispatcher`, `channels_recovery`, `subagent_maintenance`, and `self_evolution`.
- Tests: `pytest`.
- Logging: `loguru` with helpers in `clawlite/utils/logging.py`.
- CI covers Python 3.10 and 3.12.

## Important Paths
- `clawlite/cli/` - commands, ops helpers, onboarding.
- `clawlite/gateway/` - FastAPI server and WebSocket endpoints.
- `clawlite/core/` - engine, prompt builder, memory, skills, subagents.
- `clawlite/runtime/` - autonomy, supervisor, self-evolution, runtime-side telemetry.
- `clawlite/scheduler/` - cron and heartbeat services.
- `clawlite/providers/` - provider registry, discovery, failover, LiteLLM glue.
- `clawlite/channels/` - channel adapters.
- `clawlite/tools/` - tool registry and tool implementations.
- `tests/` - pytest suite mirroring runtime modules.
- `scripts/` - smoke and release-preflight helpers.
- `docs/` - operational and API docs; update when behavior changes.

## Current Runtime Notes
- `clawlite/gateway/server.py` is the control-plane center for lifecycle, diagnostics, runtime loop startup/shutdown, and supervisor recovery wiring.
- `docs/API.md` must be updated whenever diagnostics payloads change. Current additive runtime payloads include `channels_dispatcher`, `channels_recovery`, `subagents`, and `self_evolution` runner telemetry.
- `tests/gateway/test_server.py` is the main regression file for gateway diagnostics, startup replay, and supervisor recovery behavior.
- `tests/channels/test_manager.py` covers dispatcher/recovery loop diagnostics and restart behavior for the `ChannelManager`.
- The autonomy loop already includes provider-aware suppression/backoff and a repeated-idle no-progress guard; preserve those semantics when extending autonomy.
- Config parsing must preserve explicit zero values (`0`, `0.0`) for cooldown/interval settings; avoid `raw or default` when `0` is a valid input.
- When testing startup replay/autonomy notices, disable the supervisor in that test if the expected first `channels.send(...)` call must come from replay instead of a recovery notice.
- For OpenClaw parity work, adapt behavior to ClawLite architecture; do not blindly copy files or structure from another repo.

## Setup
Recommended local setup:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -e .
```

If the repo later gains dev extras, prefer:

```bash
pip install -e ".[dev]"
```

## Build, Lint, and Test Commands
There is no dedicated build script in the repo; the normal loop is editable install + lint/test/smoke.

- Install editable: `pip install -e .`
- CLI help: `python -m clawlite.cli --help`
- Start gateway: `clawlite gateway --host 127.0.0.1 --port 8787`
- Alternate start: `clawlite start --host 127.0.0.1 --port 8787`
- Validate config: `clawlite validate config`
- Release preflight: `clawlite validate preflight --gateway-url http://127.0.0.1:8787`
- Scripted preflight: `bash scripts/release_preflight.sh --config ~/.clawlite/config.json --gateway-url http://127.0.0.1:8787`
- Smoke test: `bash scripts/smoke_test.sh`

Lint commands used by the repo today:
- CI baseline: `ruff check clawlite/ --select E9,F --ignore F401,F811`
- Runtime self-check baseline: `python -m ruff check --select=E,F,W .`

Testing commands used by the repo today:
- Full suite: `python -m pytest tests -q --tb=short`
- Coverage: `python -m pytest tests/ -q --tb=short --cov=clawlite --cov-report=term-missing --cov-report=xml:coverage.xml`
- Gateway/autonomy contract slice: `python -m pytest -q tests/runtime/test_autonomy_actions.py tests/gateway/test_server.py`

Running a single test:
- Single file: `python -m pytest tests/gateway/test_server.py -q --tb=short`
- Single test: `python -m pytest tests/gateway/test_server.py::test_name -q --tb=short`
- Keyword filter: `python -m pytest tests -q --tb=short -k heartbeat`
- Fast iteration: `python -m pytest tests/path/test_file.py -q --tb=short -x`

## Validation Expectations
- For small edits, run the narrowest relevant pytest target first.
- For cross-module changes, run the touched test file plus nearby integration coverage.
- For gateway, onboarding, scheduler, or provider work, finish with `python -m pytest tests -q --tb=short` when feasible.
- If CLI or gateway behavior changes, also run a smoke command such as `clawlite --help`, `clawlite start`, or `bash scripts/smoke_test.sh`.
- If you cannot validate, say exactly what remains unverified.

## Formatting and File Style
From `.editorconfig`:
- Use UTF-8 and LF line endings.
- Insert a final newline.
- Use spaces, not tabs, in Python files.
- Default indent is 4 spaces.
- YAML/JSON/TOML use 2-space indentation.

There is no repo-wide autoformatter config checked in.
Do not do large formatting-only rewrites unless requested.

## Python Style Conventions
- Start Python modules with `from __future__ import annotations`; this is standard across the package.
- Group imports as stdlib, third-party, then local imports, with blank lines between groups.
- Prefer built-in generics like `list[str]`, `dict[str, Any]`, and `str | None`.
- Use `Path` over raw strings for filesystem work.
- Prefer small helper functions for normalization, masking, parsing, and payload shaping.
- Prefer dataclasses with `slots=True` for structured state/config objects.
- Keep public function signatures typed.
- Use Pydantic only where the repo already uses it at boundaries; keep compatibility with Pydantic v1.
- Keep constants in `UPPER_SNAKE_CASE` and private helpers prefixed with `_`.

## Naming
- Functions, methods, variables: `snake_case`.
- Classes, dataclasses, exceptions: `PascalCase`.
- Constants: `UPPER_SNAKE_CASE`.
- Test names: `test_<behavior>_<expected_result>`.
- Prefer explicit names over abbreviations.

## Error Handling
- Validate inputs early and fail fast.
- Raise `ValueError` for invalid caller input or unsupported options.
- Raise `RuntimeError` for runtime, provider, or integration failures when there is no better domain exception.
- Use `HTTPException` only at FastAPI boundaries.
- Preserve machine-readable error strings when the module already uses them, e.g. `unsupported_provider:x`.
- Chain wrapped exceptions with `raise ... from exc` when preserving cause matters.
- Never leak secrets, authorization headers, or raw tokens in errors.

## Logging
- Prefer `bind_event(...)` for structured repo logging.
- Use `logger` directly only when the module already follows that pattern.
- Use Loguru placeholder formatting, not f-strings inside logging calls.
- Include stable context like session, channel, tool, and run when available.
- Keep logs operational and scrubbed.

## Async and Background Work
- Keep async APIs async end-to-end when possible.
- Do not block the event loop with heavy sync I/O.
- Use `asyncio.to_thread(...)` for blocking work when needed.
- Be careful with long-lived loops in gateway, scheduler, and channel code.
- Respect existing timeout and cancellation behavior.

## Config and Schema
- Configuration is primarily dataclass-driven in `clawlite/config/schema.py`.
- `from_dict(...)` constructors should stay tolerant of missing keys.
- Preserve existing alias patterns when present, including snake_case and camelCase inputs.
- Do not rename or remove config fields without updating docs and tests.

## Testing Style
- Use plain pytest functions unless a class is clearly helpful.
- Common fixtures/patterns: `tmp_path`, `monkeypatch`, `capsys`, fake providers, fake clients.
- Keep tests deterministic and offline by default.
- Stub network/provider behavior instead of calling real services in unit tests.
- Assert on structured payloads and masked secrets, not only status codes.
- When fixing a bug, add or update the focused regression test first if practical.

## Change Scope and Docs
- Update docs when CLI commands, API routes, validation flow, or operational behavior changes.
- Keep one logical change per patch when possible.
- Do not silently change gateway contracts or config semantics.
- Never commit secrets, tokens, or private keys.
- Do not remove existing behavior unless the user asked for it or the replacement is clearly safe.

## Workflow Recommendations
- Read the touched module and its corresponding tests before editing.
- Search for adjacent patterns and copy the established approach.
- Prefer minimal diffs over clever rewrites.
- Ignore unrelated dirty-worktree changes unless they block the task.
- Report milestones for longer work: start, progress, completion, blocker.

When in doubt, match the surrounding code instead of introducing a new style.

---

## Commit & Pull Request Guidelines

- Follow concise, action-oriented commit messages (e.g., `channels: add Email IMAP receive`, `cli: add memory-doctor command`).
- Group related changes; avoid bundling unrelated refactors in one commit.
- Create commits scoped to your changes only; when asked to commit all, group in logical chunks.
- PR body: use `-F - <<'EOF'` heredoc for multiline bodies so newlines survive shell escaping.
- PR submission: reference the issue number as plain `#123` (no backticks) for auto-linking.
- PR landing comments: make commit SHAs clickable with full commit links.
- Changelog: user-facing changes only; no internal/meta notes.
- Pure test additions generally do **not** need a changelog entry unless they alter user-facing behavior.
- Before opening a PR, always run the full test suite: `python -m pytest tests -q --tb=short`.

## Shorthand Commands

- `sync`: if working tree is dirty, commit all changes with a sensible message, then `git pull --rebase`; if rebase conflicts and cannot resolve, stop; otherwise `git push`.
- `preflight`: run `clawlite validate preflight` + full pytest suite and report results.
- `doctor`: run `clawlite validate config && clawlite validate channels && clawlite validate onboarding` to diagnose config issues.

## GitHub Search (`gh`)

- Prefer targeted keyword search before proposing new work or duplicating fixes.
- Use `--repo <owner>/ClawLite` + `--match title,body` first; add `--match comments` when triaging follow-up threads.
- PRs: `gh search prs --repo <owner>/ClawLite --match title,body --limit 50 -- "<keyword>"`
- Issues: `gh search issues --repo <owner>/ClawLite --match title,body --limit 50 -- "<keyword>"`
- Structured output example:
  ```bash
  gh search issues --repo <owner>/ClawLite --match title,body --limit 50 \
    --json number,title,state,url,updatedAt -- "keyword" \
    --jq '.[] | "\(.number) | \(.state) | \(.title) | \(.url)"'
  ```
- Do not limit yourself to the first page; keep paginating when searching exhaustively.

## Multi-Agent Safety

- Do **not** create/apply/drop `git stash` entries unless explicitly requested (other agents may be working).
- When asked to "push", you may `git pull --rebase` to integrate latest changes; never discard other agents' work.
- When asked to "commit", scope to your changes only. When asked to "commit all", group in logical chunks.
- Do **not** switch branches unless explicitly requested.
- Do **not** create/remove `git worktree` checkouts unless explicitly requested.
- Running multiple agents is OK as long as each agent has its own session key.
- When you see unrecognized files in the working tree, keep going; focus on your changes.
- Lint/format churn: if staged+unstaged diffs are formatting-only, auto-resolve without asking. Only ask when changes are semantic (logic/data/behavior).
- Focus reports on your edits; end with a brief "other files present" note only if relevant.

## Self-Improvement Loop

ClawLite has the full toolchain for autonomous self-improvement. When asked to improve itself:

1. **Understand** — use `exec` + `files` tools to read the relevant module and its tests.
2. **Search** — use `web` tool to research best practices or OpenClaw/Nanobot implementations.
3. **Plan** — write the plan in `ROADMAP.md` or as a GitHub Issue before touching code.
4. **Implement** — edit files using `files` + `apply_patch` tools.
5. **Test** — run `python -m pytest tests -q --tb=short -x`; only proceed if green.
6. **Commit** — `git add -p` scoped to your changes + `git commit`.
7. **Notify** — send a Telegram message to the owner with what was done.

Rules for self-improvement:
- Never commit if tests are red.
- Never push secrets, tokens, or private keys.
- Never change gateway contracts or config semantics silently — create a GitHub Issue first.
- Use `skills/github` tool for creating Issues and PRs.
- Use `skills/gh-issues` to track progress.
- When unsure about scope, create a draft PR and ask via Telegram instead of proceeding.

## Security Notes

- Never commit or publish real tokens, phone numbers, or live configuration values.
- Use obviously fake placeholders in docs, tests, and examples.
- Never leak secrets, authorization headers, or raw tokens in errors or logs.
- Before triaging security issues, read `SECURITY.md`.

## Troubleshooting

- Config/validation issues: `clawlite validate preflight` + `clawlite diagnostics`.
- Provider auth failures: `clawlite provider status` + `clawlite validate provider`.
- Telegram bot not responding: `clawlite validate preflight --telegram-live` to test token.
- Gateway not starting: check `~/.clawlite/` for config, run `clawlite validate config`.
- Memory corruption: `clawlite memory doctor --repair`.
- Never send streaming/partial replies to external messaging surfaces (WhatsApp, Telegram); only final replies should be delivered there.

## Agent-Specific Notes

- Vocabulary: references to "openclaw" in this repo mean the upstream reference project; ClawLite is this repo.
- Never edit `.venv/` or installed package files; reinstall with `pip install -e .` if needed.
- When adding a new `AGENTS.md` anywhere in the repo, also create a `CLAUDE.md` symlink: `ln -s AGENTS.md CLAUDE.md` (or copy on Windows).
- Connection providers: when adding a new provider, update `clawlite/providers/registry.py`, `clawlite/providers/catalog.py`, docs, and add matching tests.
- When adding channels/skills/tools, update `docs/` and the relevant `__init__.py` exports.
- Bug investigations: read source of relevant modules and all related local code before concluding; aim for high-confidence root cause.
- For OpenClaw parity work, adapt behavior to ClawLite architecture; do not blindly copy files or structure from another repo.
- Config parsing must preserve explicit zero values (`0`, `0.0`) for cooldown/interval settings; avoid `raw or default` when `0` is a valid input.
- The autonomy loop already includes provider-aware suppression/backoff and a repeated-idle no-progress guard; preserve those semantics when extending autonomy.

---
> Source: [eobarretooo/ClawLite](https://github.com/eobarretooo/ClawLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
