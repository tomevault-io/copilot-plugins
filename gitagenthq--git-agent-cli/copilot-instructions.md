## git-agent-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build / test / install (preferred — use Makefile targets)
make build        # dev build, no credentials
make test         # all tests with -count=1 (no cache)
make install      # install to $GOPATH/bin

# Build with embedded credentials (reads from .env)
bash scripts/build.sh

# Run a single package
go test ./application/...

# Run a single test by name
go test ./application/... -run TestCommitService_NoStagedChanges

# Format all Go files (auto-runs on edit via hook; no golangci-lint configured)
gofmt -w ./...
```

**e2e tests**: `TestMain` builds the `git-agent` binary once, then all tests invoke it as a subprocess. After any source change, re-run `go test ./e2e/...` — the stale binary will not reflect changes.

## Architecture

Clean Architecture with strict inward dependency flow:

```
cmd → application → domain ← infrastructure
```

- **`domain/`** — pure Go, zero external imports; interfaces and value objects only
- **`application/`** — orchestration services (`CommitService`, `InitService`, `ScopeService`, `GitignoreService`)
- **`infrastructure/`** — adapters: git CLI wrappers, OpenAI client, config resolver, Toptal API client
- **`cmd/`** — Cobra wiring only, no business logic
- **`pkg/errors/`** — typed exit codes (0 = success, 1 = general error, 2 = hook blocked commit)
- **`e2e/`** — full binary tests via subprocess

## Key Design Decisions

**Hook dispatch**: driven by `hook` in `.git-agent/config.yml` (legacy `hook_type` in `project.yml` is auto-migrated). `""` or `"empty"` → skip validation entirely. `"conventional"` → Go-native `ValidateConventional` only (not the `conventional.sh` shell script). Any other value → Go validation first, then treat as a file path and run via shell. Shell hooks receive a JSON payload on stdin; exit 0 = allow, non-zero = block. After 3 retries, `git-agent` exits with code 2.

**Multi-commit flow**: for each planned `CommitGroup`, `CommitService` calls `git.UnstageAll()` then `git.StageFiles(group.Files)` before generating and committing. Hook failures after 3 retries trigger a re-plan of remaining files (capped at 2 re-plans). If any group title lacks a scope, scopes are refreshed and the plan is regenerated once.

**Amend flow**: `--amend` calls `LastCommitDiff()`, generates a new message, and calls `AmendCommit()`. No planning or hook execution.

**Auto-scope**: if `CommitRequest.Config` is nil or has no scopes, `CommitService` calls `ScopeService.Generate()` and `MergeAndSave()` automatically before planning. Pass `Config: &project.Config{}` (non-nil, empty) to suppress this.

**Config precedence**: CLI flag > `~/.config/git-agent/config.yml` > zero-config default. Project config (`.git-agent/config.yml`) provides scopes, hooks, and behavior flags — credentials never go there. Local overrides in `.git-agent/config.local.yml` take precedence over project config.

**Trailer handling**: trailers are assembled in `cmd/commit.go` and appended via `git interpret-trailers` before each `git.Commit()`. On hook retry, `previousMessage = preTrailer` (title + body without trailers) so trailers never enter LLM context.

## Commit Conventions

Enforced via pre-tool hook. Commit messages must:
- Title: `type(scope): description` — all lowercase, ≤50 characters
- Valid scopes: `docs`, `plans`, `design`, `cli`
- Body: bullet points (imperative verbs) + closing explanation paragraph
- Body lines ≤72 characters

---
> Source: [GitAgentHQ/git-agent-cli](https://github.com/GitAgentHQ/git-agent-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
