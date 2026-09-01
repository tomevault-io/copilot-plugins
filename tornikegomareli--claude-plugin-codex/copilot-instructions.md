## claude-plugin-codex

> A Codex CLI plugin that delegates work to the local Claude Code CLI — the mirror image of OpenAI's `codex-plugin-cc`. Codex users invoke it by mention (`@claude review`, `@claude plan`, `@claude implement`, `@claude rescue`, `@claude transfer`, ...) or as `/claude:<name>` in `codex exec`; each command runs Claude Code headlessly and manages background jobs.

# claude-plugin-codex

A Codex CLI plugin that delegates work to the local Claude Code CLI — the mirror image of OpenAI's `codex-plugin-cc`. Codex users invoke it by mention (`@claude review`, `@claude plan`, `@claude implement`, `@claude rescue`, `@claude transfer`, ...) or as `/claude:<name>` in `codex exec`; each command runs Claude Code headlessly and manages background jobs.

## Layout

```
.agents/plugins/marketplace.json   # Codex marketplace manifest (marketplace name: claude-code)
plugins/claude/                    # the plugin itself
  .codex-plugin/plugin.json        # plugin manifest (version synced by scripts/bump-version.mjs)
  commands/*.md                    # slash commands; each shells out to the companion script
  agents/claude-rescue.md          # thin forwarder agent for @claude rescue
  skills/                          # claude-cli-runtime, claude-result-handling, claude-prompting
  prompts/*.md                     # {{VARIABLE}} templates interpolated by lib/prompts.mjs
  schemas/review-output.schema.json# structured review findings schema
  scripts/claude-companion.mjs     # single CLI entrypoint; all verbs dispatch from here
  scripts/lib/                     # runtime modules
  scripts/hooks/                   # Codex SessionStart + Stop hook scripts
tests/                             # node:test suites + fake `claude` binary fixture
scripts/bump-version.mjs           # fans package.json version to plugin.json; --check in CI
```

## Development

- Run tests: `npm test` (node --test, no network, no real claude — everything goes through `tests/fake-claude-fixture.mjs`)
- Version bump: `node scripts/bump-version.mjs <version>`; CI runs `--check`
- Zero runtime dependencies; plain Node ESM (`.mjs`), Node >= 18.18
- Local install for manual testing: `codex plugin marketplace add <repo-dir>` then `codex plugin add claude@claude-code`

## Architecture

- `scripts/lib/claude.mjs` — `runClaudeTurn()` spawns `claude -p --output-format stream-json --verbose`, feeds the prompt via stdin, and parses NDJSON events (`lib/claude-stream.mjs`) into progress phases (starting/investigating/editing/verifying/finalizing) and the final result (`resultText`, `structured_output`, `session_id`, cost).
- Permission model: reviews and plans run with `--permission-mode plan` (read-only); write tasks use `acceptEdits` plus a curated `--allowedTools` list; `--full-auto` maps to `--dangerously-skip-permissions`.
- Jobs: every verb runs through `runTrackedJob` (lib/tracked-jobs.mjs); state lives in `<data-root>/state/<slug>-<hash>/` as `state.json` + per-job `<id>.json`/`.log`. Background runs spawn a detached `job-worker` subprocess. Resume uses the stored `claudeSessionId` with `claude --resume`.
- Data root resolution (lib/state.mjs): `CLAUDE_COMPANION_DATA` env → `~/.codex/claude-companion` if writable → tmpdir. The writability probe matters: Codex sandboxes often cannot write under `~/.codex`.
- Transfer (lib/codex-rollout.mjs, codex-session-transfer.mjs, claude-session-writer.mjs): parses a Codex rollout, flattens each turn into plain-text user/assistant records (tool calls become inline bullets; thinking/tool blocks cannot be synthesized because they carry signatures), writes `~/.claude/projects/<sanitized-cwd>/<uuid>.jsonl`, and smoke-tests `claude --resume` before publishing.
- Hooks: Codex plugins cannot ship hooks, so `@claude setup --enable-review-gate` writes entries into `~/.codex/hooks.json` via lib/hooks-config.mjs (marker `claude-plugin-codex`, idempotent merge, never clobbers foreign entries). The Stop hook runs a plan-mode review of the previous Codex turn and blocks with `{"decision":"block"}` on `BLOCK:` output; `stop_hook_active` guards against loops.

## Gotchas

- Claude's `--json-schema` silently returns no structured output if the schema contains a `$schema` key — `readOutputSchema()` strips it. It also needs spare turns; never combine it with `--max-turns 1`.
- `state.json` writes must stay atomic (tmp + rename) and `runClaudeTurn`'s SIGTERM handler must exit the process — both guard the cancel race where a killed worker resurrects or corrupts job state.
- Claude session dir sanitization replaces every char outside `[A-Za-z0-9-]` with `-` (including `.` and `_`).
- Codex rollout `token_count` usage lives under `payload.info.last_token_usage`.
- Command markdown must not reference Claude-Code-only concepts (AskUserQuestion, run_in_background, CLAUDE_PLUGIN_ROOT) — `tests/commands.test.mjs` enforces this.

## Testing conventions

- `tests/fake-claude-fixture.mjs` installs a fake `claude` binary that records the full argv (`lastInvocation`) and emits NDJSON; behaviors select scenarios (`review-ok`, `slow-task`, `block-gate`, ...). Assert flag propagation via the recorded argv.
- Point `CLAUDE_COMPANION_DATA` and `CODEX_HOME` at temp dirs in every test; never touch the real `~/.codex` or `~/.claude`.

---
> Source: [tornikegomareli/claude-plugin-codex](https://github.com/tornikegomareli/claude-plugin-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
