## wt

> This file provides guidance for Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance for Claude Code when working with this repository.

## Project Overview

`wt` is a CLI tool that manages git worktrees and tmux sessions. It provides seamless integration between git bare repositories with worktrees and tmux, allowing developers to quickly switch between branches with their own isolated tmux sessions.

## Build & Test Commands

```bash
make build      # Build binary to ./build/wt
make install    # Build and install to ~/.local/bin/wt (includes codesign for macOS)
make test       # Run all tests
make clean      # Remove build artifacts
make fmt        # Format code
make lint       # Run linter (requires golangci-lint)

# Test with coverage
go test -cover ./...

# Verbose test output
go test -v ./...
```

## Project Structure

```
.
├── cmd/wt/main.go           # Entry point
├── internal/
│   ├── cli/                 # Cobra commands (add, delete, switch, toggle, etc.)
│   ├── config/              # YAML config loading/merging (global + project)
│   ├── files/               # Symlink/copy operations for shared files
│   ├── git/                 # Git bare repo and worktree operations
│   ├── tmux/                # Tmux session, window, pane management
│   └── tui/                 # Bubbletea TUI picker for interactive selection
├── Makefile
└── .wt.yaml                 # Example project config
```

## Key Concepts

### Bare Repository Structure
The tool expects git bare repositories where worktrees are created inside:
```
project.git/           # Bare repo (GitDir)
├── main/              # Worktree for main branch
├── feature-x/         # Worktree for feature-x branch
├── .wt.yaml           # Project config (optional)
└── [git internals]
```

### Config Hierarchy
1. **Global config**: `~/.config/wt/config.yaml`
2. **Project config**: `.wt.yaml` in bare repo root
3. Project overrides global, with `<global>` marker for merging lists

### Toggle Pane
A special pane (e.g., Claude) that can be toggled in any window:
- Hidden in window `_claude` when not visible
- Joined to current window when toggled on
- Identified by pane title `wt-toggle-pane` for reliable show/hide

## Code Patterns

### Error Handling
- Return wrapped errors with context: `fmt.Errorf("failed to X: %w", err)`
- CLI commands print user-friendly messages, internal packages return errors

### Git Operations
- All git commands use `exec.Command("git", ...)`
- Bare repo detection via `git rev-parse --git-common-dir`
- Worktree operations via `git worktree add/remove/list`

### Tmux Operations
- Check `IsInsideTmux()` before tmux-specific operations
- Session names derived from branch names (dots replaced with underscores)
- Pane identification via titles for reliable toggle behavior

### Testing
- Use `setupTestBareRepo(t)` helper for git tests (creates temp bare repo with initial commit)
- Tests that require tmux should check `IsInsideTmux()` and skip if false
- Config tests mock HOME/XDG_CONFIG_HOME environment variables

## Common Tasks

### Adding a New CLI Command
1. Create `internal/cli/commandname.go`
2. Define `var commandCmd = &cobra.Command{...}`
3. Add to `rootCmd` in `internal/cli/root.go` init()

### Modifying Config Structure
1. Update structs in `internal/config/config.go`
2. Update `DefaultConfig()` if needed
3. Update `mergeConfigs()` for proper global/project merging
4. Add tests in `config_test.go`

### Testing Tmux Functionality
Integration tests for tmux are skipped outside tmux. To run them:
```bash
tmux new-session -d -s test "cd $(pwd) && go test -v ./internal/tmux/..."
tmux attach -t test
```

## Important Files

- `internal/cli/root.go` - CLI initialization, config loading
- `internal/git/worktree.go` - Core worktree operations
- `internal/tmux/pane.go` - Toggle pane logic
- `internal/config/config.go` - Config structures and loading

---
> Source: [yun-jay/wt](https://github.com/yun-jay/wt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
