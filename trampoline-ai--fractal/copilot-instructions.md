## fractal

> Fractal is an interactive local coding-agent CLI built around PredictRLM. Each

# AGENTS.md

Fractal is an interactive local coding-agent CLI built around PredictRLM. Each
user turn is one RLM call over a `predict_rlm.Workspace`, with the repo mounted
in the PredictRLM sandbox and workspace edits synced back after generated code
blocks run.

## Current Shape

- CLI entry point: `fractal` -> `fractal.cli:main`.
- Main REPL: `src/fractal/cli.py`.
- Agent wrapper: `src/fractal/agent/service.py`.
- RLM signature: `src/fractal/agent/signature.py`.
- PredictRLM skill instructions: `src/fractal/agent/skills.py`.
- Session persistence: `src/fractal/session.py`.
- Smoke tests: `tests/test_smoke.py`.
- Session tests: `tests/test_session.py`.

The current implementation stores structured session summaries, bounded
PredictRLM history, and host-recorded per-turn usage (tokens, cost, duration,
iterations, live context size) under `.fractal/sessions/<session_id>.json`.
Sessions resume via `--resume <id>` or `/resume`; `/sessions` lists stored
sessions and `/usage` reports session totals. The Linear roadmap tracks the
remaining work needed to turn this into a reliable daily-use coding agent.

Observed predict-rlm issues are recorded in `docs/predict-rlm-notes.md`
instead of being worked around or fixed here.

## Local Commands

This repo uses `uv` and depends on a local editable checkout of
`../predict-rlm`.

```bash
uv run fractal --help
uv run pytest
```

The RLM-facing imports require `predict_rlm.Workspace` from the local
`/Users/emile/git/predict-rlm` checkout, currently expected on a branch that
exports workspace support.

## Product Direction

Fractal's MVP is a local interactive coding-agent CLI, not a broad cloud agent
platform. The table-stakes MVP capabilities tracked in Linear are:

- Durable structured session state.
- Deterministic context packet generation from session state.
- PredictRLM run trace ingestion and persistence.
- Safe host command execution for tests, builds, linters, and inspection.
- Command execution policy and safety boundaries.
- Reliable changed-file detection from workspace state.
- Project context and instruction-file loading.
- Standard agent skill support at the harness level.
- Interactive configuration and auth setup.
- Provider/auth abstraction for API keys and future Codex-style auth.
- Minimal turn observability and user-facing run summaries.
- Failed-turn persistence and recovery context.

Known post-MVP gaps versus current coding agents include git-native change
lifecycle support: exact diffs per turn, checkpoints, rollback, dirty-worktree
detection, branch/worktree awareness, and optional commit generation.

## Engineering Guidelines

- Keep changes narrowly scoped. This repo is small; avoid introducing broad
  abstractions before the behavior needs them.
- Commit messages must use a conventional prefix such as `feat:`, `fix:`,
  `docs:`, `tests:`, `refactor:`, `chore:`, or `improv:`.
- Fractal depends on the local editable `../predict-rlm` checkout, and we own
  that package too. If a problem is caused by a PredictRLM interface,
  serialization, sandbox, or trace limitation, call it out as a PredictRLM issue
  instead of adding a Fractal workaround by default. Fix PredictRLM directly
  when that is the right layer.
- Prefer host-side truth over model-reported truth for state, files changed,
  commands run, verification status, and errors.
- Preserve user work. Fractal should not overwrite unrelated edits or hide dirty
  workspace state.
- Add comments for non-obvious decisions, not for mechanics. Explain why code is
  shaped a certain way, especially around session state, PredictRLM prompt
  wiring, persistence, recovery, safety, and retention. Avoid comments that only
  restate what the next line of code does.
- Persist enough state to recover from failures. A failed agent turn should
  leave useful session records, error details, changed files, and verification
  context.
- Do not put `.fractal` session data into the PredictRLM workspace context; the
  agent service currently excludes it from `Workspace`.
- Keep CLI output concise but operationally useful: status, changed files,
  commands run, verification result, and next recovery step when blocked.

## Testing Guidance

- Run `uv run pytest` for normal validation.
- Tests that require `predict_rlm.Workspace` are written to skip when the local
  predict-rlm checkout does not export it.
- Add focused tests for session migration/persistence, context building, command
  policy, changed-file detection, and failure handling as those features land.
- Tests should verify behavior and durable state, not incidental presentation
  details. Avoid assertions that depend on exact terminal wrapping, whitespace,
  box-drawing characters, prompt counts, color/style internals, or the mere
  existence of static files. For CLI output, prefer checking semantic content or
  normalized text unless layout itself is the behavior under test.

## Current Caveats

- There is no host command execution tool yet.
- There is no robust approval/sandbox policy yet.
- Changed files are currently coerced from model output.
- There is no git checkpoint, diff review, or rollback layer yet.
- There is no MCP/plugin system beyond the planned standard skill loader.

---
> Source: [Trampoline-AI/fractal](https://github.com/Trampoline-AI/fractal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
