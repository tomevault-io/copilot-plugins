## prr

> - **Never read `.env`** (it contains secrets). Use `source .env && ./prr` to load env vars when running the binary.

# AGENTS.md

## Environment

- **Never read `.env`** (it contains secrets). Use `source .env && ./prr` to load env vars when running the binary.

## Build & verify

```bash
go build -o prr ./cmd/prr        # build binary
go test ./...                     # all tests
gofmt -l .                        # lint (CI fails if any output)
go vet ./...                      # vet
```

CI runs: `gofmt` + `go vet` + `go test` on every PR to `master`.

## Project layout

Single Go module. Entrypoint: `cmd/prr/main.go`.

| Package | Role |
|---------|------|
| `internal/ui` | Bubble Tea TUI (Elm architecture: Model/View/Update) |
| `internal/ai` | LLM provider adapters (Gemini, OpenAI, GitHub Copilot, Claude Code CLI), agent loop, tool execution |
| `internal/git` | Git/GitHub CLI wrappers, diff parsing, PR fetching |
| `internal/config` | Config loading, model definitions, exclusion rules |
| `internal/review` | Review logic, prompt construction, routing |
| `internal/state` | Per-PR review state persistence |
| `internal/security` | Security scanning |
| `internal/pipe` | Export targets for findings |

## Key conventions

- **Never use `fmt.Print*` for output** — stdout is owned by Bubble Tea. Use `log.Printf` for debug logging.
- **Error wrapping**: `fmt.Errorf("context: %w", err)`.
- **Formatting**: `gofmt` only (no goimports, no custom config).
- **Codegen**: `scripts/update-models.sh` regenerates `internal/config/known_models.go` from the Gemini API. Reads the API key from `~/.config/prr/config.json`. Requires `python3`. Do not hand-edit that file.
- **Claude Code provider** (`internal/ai/claudecode.go`): shells out to `claude -p` per request. Unlike the other providers, it does **not** drive prr's tool loop — it runs its own internal loop with a curated read-only toolset (Read/Grep/Glob + specific read-only git subcommands), and only the final text/thinking is surfaced as `ChatEvent`s. Detection is a `exec.LookPath("claude")` registered into `config.KeylessProviderAvailable` at init.

## Testing

- Integration tests require `PRR_LIVE_TESTS=1` and a valid `~/.config/prr/config.json` with API keys configured. They are skipped otherwise.
- Run live tests: `PRR_LIVE_TESTS=1 go test ./internal/ai/ -run TestLive -v`
- Helper script: `scripts/test-live.sh` (sets `PRR_LIVE_TESTS=1` automatically).

## Branching & release

- Default branch: `master`.
- Releases: automated via release-please + goreleaser on `master`.

## RULES

These rules apply to every task in this project unless explicitly overridden.
Bias: caution over speed on non-trivial work. Use judgment on trivial tasks.

## Rule 1 — Think Before Coding
State assumptions explicitly. If uncertain, ask rather than guess.
Present multiple interpretations when ambiguity exists.
Push back when a simpler approach exists.
Stop when confused. Name what's unclear.

## Rule 2 — Simplicity First
Minimum code that solves the problem. Nothing speculative.
No features beyond what was asked. No abstractions for single-use code.
Test: would a senior engineer say this is overcomplicated? If yes, simplify.

## Rule 3 — Surgical Changes
Touch only what you must. Clean up only your own mess.
Don't "improve" adjacent code, comments, or formatting.
Don't refactor what isn't broken. Match existing style.

## Rule 4 — Goal-Driven Execution
Define success criteria. Loop until verified.
Don't follow steps. Define success and iterate.
Strong success criteria let you loop independently.

## Rule 5 — Use the model only for judgment calls
Use me for: classification, drafting, summarization, extraction.
Do NOT use me for: routing, retries, deterministic transforms.
If code can answer, code answers.

## Rule 6 — Token budgets are not advisory
Per-task: 4,000 tokens. Per-session: 30,000 tokens.
If approaching budget, summarize and start fresh.
Surface the breach. Do not silently overrun.

## Rule 7 — Surface conflicts, don't average them
If two patterns contradict, pick one (more recent / more tested).
Explain why. Flag the other for cleanup.
Don't blend conflicting patterns.

## Rule 8 — Read before you write
Before adding code, read exports, immediate callers, shared utilities.
"Looks orthogonal" is dangerous. If unsure why code is structured a way, ask.

## Rule 9 — Tests verify intent, not just behavior
Tests must encode WHY behavior matters, not just WHAT it does.
A test that can't fail when business logic changes is wrong.

## Rule 10 — Checkpoint after every significant step
Summarize what was done, what's verified, what's left.
Don't continue from a state you can't describe back.
If you lose track, stop and restate.

## Rule 11 — Match the codebase's conventions, even if you disagree
Conformance > taste inside the codebase.
If you genuinely think a convention is harmful, surface it. Don't fork silently.

## Rule 12 — Fail loud
"Completed" is wrong if anything was skipped silently.
"Tests pass" is wrong if any were skipped.
Default to surfacing uncertainty, not hiding it.

---
> Source: [andreujuanc/prr](https://github.com/andreujuanc/prr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
