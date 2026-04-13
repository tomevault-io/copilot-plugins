## wt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`wt` is a Go CLI for managing git worktrees in a bare repo setup. It wraps git commands to automate cloning, worktree creation, upstream tracking, file copying, and post-create scripts.

## Development

```sh
make build         # build the `wt` binary
make test          # run all tests
make install       # install to $GOPATH/bin
```

Only dependency is `github.com/spf13/cobra` for CLI routing.

```sh
go test ./...      # run all tests
```

## Architecture

Entry point is `main.go` which uses `cobra` to register eight subcommand handlers.

**Shared packages:**
- `internal/git/git.go` — two helpers: `Git(args, opts)` for general git calls, `GitInBare(args, projectRoot)` for commands targeting the `.bare/` directory. Both use `os/exec`.
- `internal/config/config.go` — finds `worktree.json` by walking up from cwd, loads/writes it. `LoadConfig()` returns `(root, *Config, error)`.
- `internal/cmd/deps.go` — `Deps` struct with `GitRunner` interface, `ConfigLoader` interface, `Prompter` interface, and `io.Writer` fields for stdout/stderr. Enables dependency injection for testing.

**Commands (`internal/cmd/`):**
- `init.go` — interactive setup: clones bare repo into `.bare/`, creates `.git` file pointing to it, detects default branch, writes `worktree.json`
- `create.go` — fetches, creates worktree + branch from `<remote>/<base>`, copies files, runs post-create script. Exports `SetupWorktree()` for shared post-creation logic.
- `pr.go` — creates a worktree for a PR under `prs/<number>/`, using `gh` CLI to resolve the branch name
- `list.go` — thin wrapper around `git worktree list`
- `remove.go` — removes worktree, optionally deletes branch (with interactive confirmation)
- `prune.go` — bulk-removes worktrees whose branches are merged into `<remote>/<defaultBase>`
- `cd.go` — resolves a worktree name to an absolute path (exact branch, basename, relative path, or substring match)
- `shellinit.go` — outputs a shell function wrapper so `wt cd` can change the parent shell's directory

**Key patterns:**
- All commands use `Deps` for dependency injection — tests provide mock implementations.
- Commands use Cobra's `RunE`, returning errors instead of calling `os.Exit`.
- User prompts use `bufio.Scanner` via the `Prompter` interface.

## Workflow

When adding, removing, or changing commands, always update `README.md` to reflect the change.

## Config

Projects are identified by `worktree.json` at the project root. Fields: `remote`, `defaultBase`, `copyFiles` (array), `postCreateScript`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gsigler)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gsigler)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
