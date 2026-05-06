## codex-loop

> `codex-loop` is a local-first Go CLI and Codex plugin that keeps explicitly activated Codex tasks running until they satisfy either a minimum duration or a target number of deliberate rounds.

# codex-loop Agent Instructions

## Project Overview

`codex-loop` is a local-first Go CLI and Codex plugin that keeps explicitly activated Codex tasks running until they satisfy either a minimum duration or a target number of deliberate rounds.

The project ships as:

- a Go single-binary CLI at `cmd/codex-loop`;
- internal runtime packages under `internal/`;
- a Codex plugin bundle under `plugins/codex-loop`;
- a local marketplace entry under `.agents/plugins/marketplace.json`.

Core product premise: every feature must remain usable from the CLI and from Codex lifecycle hooks. Features are incomplete if they only work through internal Go calls.

## Most Critical Rules

### Git Commands Restriction

- **Never run destructive git commands without explicit user permission.**
- Forbidden without explicit permission: `git restore`, `git checkout`, `git reset`, `git clean`, `git rm`.
- Do not restore, remove, or rewrite files that are unrelated to your task.
- If `git status` shows unexpected changes, assume they belong to the user or another agent. Read around them and work with them.

### Plan Mode Persistence

- In Plan mode, after the user accepts a plan, always write the accepted plan to `.codex/plans/`.
- If the accepted plan changes later, update or append the corresponding file under `.codex/plans/`.

### Test Integrity

- Tests exist to discover bugs, not to create mock-driven confidence.
- When a test reveals unexpected behavior, fix production code instead of weakening the assertion.
- Mocks and stubs may be used only at I/O boundaries in unit tests. Final validation for meaningful behavior must include real integration, CLI, hook, filesystem, or end-to-end style checks where applicable.

## Memory Ledger

Maintain one Memory Ledger per agent session in `.codex/ledger/<YYYY-MM-DD>-MEMORY-<slug>.md`.

At the start of every assistant turn:

- read your own ledger file if it exists;
- scan other `*-MEMORY-*.md` files in `.codex/ledger/` for cross-agent awareness;
- treat other agents' ledgers as read-only.

Update your own ledger whenever the goal, constraints, decisions, state, progress, or important tool outcomes change. Keep it short, factual, and compaction-safe. Mark uncertainty as `UNCONFIRMED`.

Use this format:

```markdown
Goal (incl. success criteria):
Constraints/Assumptions:
Key decisions:
State:
Done:
Now:
Next:
Open questions (UNCONFIRMED if needed):
Working set (files/ids/commands):
```

In replies, begin with a brief Ledger Snapshot: Goal, Now/Next, and Open Questions.

## Greenfield OSS v1

- Treat this project as the public `codex-loop` implementation.
- Do not add compatibility aliases, migration bridges, fallback schemas, or references to prior project names unless the user explicitly asks.
- Renames must update code, plugin metadata, docs, tests, marketplace entries, and examples in the same change.
- Delete obsolete code instead of preserving unused compatibility paths.

## Critical Engineering Rules

- `make verify` must pass before completing any code, plugin, or behavior task.
- For Go changes, run `go vet ./...` before or as part of the final gate.
- `make lint` is strict `golangci-lint`; zero warnings and zero issues are acceptable.
- Never add dependencies by editing `go.mod` by hand. Use `go get`, then `go mod tidy`.
- Never use web search for local project code. Use `rg`, `rg --files`, `find`, and local file reads.
- Use external docs only for external APIs, libraries, or current platform behavior. For Codex/OpenAI plugin or hook behavior, prefer official OpenAI documentation.
- Never ignore errors with `_` in production code or tests unless a short written justification is next to the discard.
- Do not commit local scratch, QA, or runtime artifacts such as `.tmp/`, ad hoc Codex homes, coverage files, or generated test sandboxes unless the task explicitly requires them.
- Conversation with the user may be in Brazilian Portuguese. Durable artifacts such as code, comments, docs, plans, commit messages, and this file must be in English.

## Skill Dispatch

Activate skills before writing code or durable project instructions. Use the smallest set that covers the task.

| Domain | Required Skills | Conditional Skills |
| --- | --- | --- |
| Go runtime, CLI, hooks, installer | `golang-pro` | `context7`, `find-docs` |
| Concurrency, cancellation, race fixes | `golang-pro` | `architectural-analysis` |
| Go tests | `golang-pro` | `qa-report`, `qa-execution` |
| Codex plugin metadata, lifecycle hooks, OpenAI behavior | `openai-docs` | `context7`, `find-docs` |
| Release or scenario QA | `qa-report`, `qa-execution` | `golang-pro` |
| Architecture audit, dead code, duplication | `architectural-analysis` | `golang-pro` |
| TUI work, if ever introduced | `bubbletea`, `golang-pro` | |
| External documentation lookup | `context7` or `find-docs` | `exa-web-search-free` |

Every Go change requires `golang-pro`, even when the edit is small.

## Golang Pro Requirements

Follow `golang-pro` as the baseline for all Go work:

- design small interfaces first when behavior crosses package boundaries;
- pass `context.Context` to blocking, filesystem, subprocess, hook, install, or future network operations;
- manage every goroutine with a clear lifecycle, cancellation path, and error propagation path;
- handle all errors explicitly and wrap with `fmt.Errorf("...: %w", err)` when returning context;
- use `errors.Is` and `errors.As` for classified error handling;
- avoid `panic` for normal error handling;
- avoid reflection unless there is a measured reason;
- use gofmt for all Go files;
- run `go vet ./...`, `golangci-lint`, race-enabled tests, and build checks before handoff;
- write table-driven tests with subtests for parser, hook, CLI, installer, and store behavior;
- document exported packages, types, functions, and methods;
- use Go union constraints such as `X | Y` for generic constraints when generics are needed;
- prefer functional options or explicit config structs over hardcoded configuration.

## Build Commands

Use the Makefile targets. They delegate to Mage and install Mage on demand when needed.

```bash
make deps       # go mod tidy
make fmt        # gofmt all non-hidden Go files
make vet        # go vet ./...
make lint       # golangci-lint v1.64.8 run ./...
make test       # go test -race ./...
make build      # go build ./...
make verify     # fmt -> vet -> lint -> test -> build
make help       # list Mage targets
```

Run narrower commands while iterating, but the final gate for project changes is `make verify`.

## Surface Map

| Path | Responsibility |
| --- | --- |
| `cmd/codex-loop` | Binary entry point only. Keep business logic out of `main`. |
| `internal/cli` | Cobra commands, flags, stdin/stdout/stderr wiring, status JSON command behavior. |
| `internal/loop` | Activation parsing, hook processing, loop state, config, continuation decisions. |
| `internal/installer` | `CODEX_HOME` handling, runtime install/uninstall, config preservation. |
| `internal/logger` | Logging helpers. Keep hook stdout clean for protocol JSON. |
| `internal/pluginmeta` | Plugin and marketplace metadata validation tests. |
| `internal/version` | Version metadata. |
| `plugins/codex-loop` | Codex plugin bundle: manifest, lifecycle hooks, user-facing skill. |
| `.agents/plugins/marketplace.json` | Local marketplace entry pointing to `./plugins/codex-loop`. |
| `.codex/plans` | Accepted plans only. |
| `.codex/ledger` | Agent session memory, usually not product documentation. |

## Product Invariants

- Activation only happens when the first prompt line is a valid `[[CODEX_LOOP ...]]` header.
- `name="..."` is required.
- Exactly one limiter is required: `min="..."` or `rounds="..."`.
- `rounds` must be a positive integer.
- Supported durations include forms such as `30m`, `30min`, `1h 30m`, `2 hours`, and `45sec`.
- Loop state is isolated by Codex `session_id`.
- State persists under `${CODEX_HOME:-$HOME/.codex}/codex-loop/loops/`.
- State writes must be atomic and leave a debuggable JSON shape.
- `status` output is JSON and should stay stable unless the change is intentional and documented.
- `codex-loop install` may create or update managed `codex-loop` runtime files and enable `features.codex_hooks`.
- `codex-loop uninstall` removes only managed `${CODEX_HOME}/codex-loop/` artifacts and must preserve unrelated Codex configuration and plugin state.
- Plugin lifecycle hooks come from `plugins/codex-loop/hooks/hooks.json`; do not make manual edits to global hook files the primary integration path.
- The project is local-first. Do not add network behavior, telemetry, or external services without an explicit product decision.

## Coding Style

- Keep packages small and explicit. Avoid package-level mutable state unless it is immutable configuration or version metadata.
- Keep `main` thin: construct the CLI, execute it, and exit.
- Cobra commands must be testable with injected args, streams, env, and filesystem roots. Do not call `os.Exit` from internal packages.
- Keep hook handlers protocol-safe: hook JSON goes to stdout; logs and diagnostics go to stderr or files.
- Preserve user configuration. Installer code must update only the keys and paths it owns.
- Prefer typed structs and JSON/TOML encoders over string concatenation for structured data.
- Use `filepath` and `os.UserHomeDir` style APIs for paths. Do not assume Unix-only separators outside shell snippets.
- Keep shell embedded in plugin metadata minimal, quoted, and tolerant when the runtime binary is not installed yet.
- Add comments only when they explain non-obvious behavior, protocol constraints, or failure modes.

## Testing

- Default to table-driven tests with subtests.
- Use `t.TempDir()` for filesystem tests.
- Use isolated `CODEX_HOME` values in installer, hook, and status tests. Never point tests at the user's real `~/.codex`.
- Use real files and real JSON/TOML parsing for config, store, installer, and plugin metadata tests.
- Assert both success output and failure behavior. Status-code-only or error-only checks are insufficient when body/JSON content matters.
- Keep race-sensitive tests compatible with `go test -race ./...`.
- Add regression tests for bugs before or with the fix.
- Do not weaken tests to match broken behavior.

## Code Search Hierarchy

1. `rg` / `rg --files` for local code.
2. Local docs, README, plans, ledgers, and plugin metadata.
3. `context7`, `find-docs`, or `openai-docs` for external technical documentation.
4. Web search only when authoritative docs are insufficient or the user explicitly asks.

## Commit Style

- Do not create commits unless the user asks.
- When committing, use exactly one prefix: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, or `build:`.
- Do not use `chore:`, `style:`, or `ci:`.
- Use `build:` for tooling, release, and CI changes.
- Run `make verify` before committing.
- If a hook or verification fails, fix the issue in a follow-up change. Do not use destructive git commands to recover.

## QA Loop Reuse

- Prefer a fresh isolated QA home for a new independent QA pass.
- Reuse an existing QA lab only when continuing the same active session and a manifest for that exact run exists.
- Never run provider-backed QA against the user's raw global `~/.codex`; isolate `CODEX_HOME`.
- Persist and report any reusable QA manifest path, lab root, runtime home, base URL if applicable, and verification evidence.

## Cross References

- Project plan: `.codex/plans/2026-04-30-codex-loop-go-cli.md`
- Plugin manifest: `plugins/codex-loop/.codex-plugin/plugin.json`
- Hook config: `plugins/codex-loop/hooks/hooks.json`
- Plugin skill: `plugins/codex-loop/skills/codex-loop/SKILL.md`
- Local marketplace: `.agents/plugins/marketplace.json`

---
> Source: [compozy/codex-loop](https://github.com/compozy/codex-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
