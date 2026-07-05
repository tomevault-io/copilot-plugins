## plankit

> IMPORTANT: Follow these rules at all times.

# CLAUDE.md

IMPORTANT: Follow these rules at all times.

## Critical Rules

- NEVER take shortcuts without asking. STOP, ASK, WAIT for approval.
- NEVER force push. Make a new commit to fix mistakes.
- NEVER commit secrets to version control.
- Only do what was asked. No scope creep.
- Understand existing code before changing it.
- If you don't know, say so. Never guess.
- Test before and after every change.
- Surface errors clearly. No silent fallbacks.
- When rules conflict, prefer the safer, more reversible action and ask.

## Project Conventions

### Branch & Workflow

- All changes go through `develop` — never commit directly to `main`.
- Release flow: `pk changelog` (on `develop`) → `pk release` (merges to `main`, pushes, switches back).
- Conventional Commits required. Configured types: `feat`, `fix`, `deprecate`, `revert`, `security`, `refactor`, `perf`, `docs`, `chore`, `test`, `build`, `ci`, `style`, `plan` (hidden).

### Quick Commands

```bash
make build          # Build for current platform -> dist/pk
make test           # Run tests with race detector
make build-all      # Cross-compile for 5 platforms
make install        # Install to GOPATH/bin
make lint           # Run go vet + gofmt drift check
make vuln           # Scan for known vulnerabilities (govulncheck)
pk changelog        # Generate CHANGELOG.md and commit (no tag)
pk changelog --undo # Unwind an unpushed release commit
pk release          # Read Release-Tag trailer, create tag, merge, and push
```

- **Always use `make build`, never `go build ./cmd/pk` directly.** Bare `go build` drops a binary in the working directory; the Makefile routes output to `dist/`.

### Language & Build

- **Go 1.26**, standard library only — no third-party dependencies.
- Binary: `pk` — single entrypoint at `cmd/pk/main.go`.
- Build: `make build` (output to `dist/`).
- Test: `make test` (runs `go test -v -race ./...`).
- Cross-compile: `make build-all` (darwin/linux amd64+arm64, windows amd64).
- Version injected via ldflags (`-X .../version.version=$(VERSION)`).
- All user messages to stderr, stdout reserved for hook protocol JSON.
- `CGO_ENABLED=0` enforced via Makefile — pure-Go static binaries, no implicit glibc dependency on linux.
- CLI flags use `--kebab-case` (e.g., `--dry-run`, `--project-dir`).

### Directory Structure

- `cmd/pk/` — CLI entrypoint, flag parsing, subcommand dispatch.
- `internal/` — all packages: `changelog`, `config`, `git`, `guard`, `hooks`, `preserve`, `protect`, `release`, `setup`, `status`, `teardown`, `update`, `version`.
- `internal/setup/` — organized by concern: `claude.go` for Claude Code-specific wiring (hooks, settings, bootstrap), `managed.go`/`pin.go`/`baseline.go` for universal logic, `setup.go` for orchestration.
- `docs/` — user-facing documentation. `docs/plans/` — preserved plans (immutable after creation).
- `.claude/skills/` — managed skills (conventions, preserve, ship).
- `.claude/rules/plankit/` — managed rules (development-standards, git-discipline, model-behavior, plankit-tooling), installed under a `plankit/` subdirectory so they never collide with a project's own `.claude/rules/` files (Claude Code discovers rules recursively). `plankit-development.md` (maintainer-only, not shipped) stays at `.claude/rules/`.
- `site/` — landing page.

### Design

- **All commands resolve to the git repository root.** A pk command can be invoked from any subdirectory; it walks up to find `.git` and operates there. Commands don't require being at the root, but they effectively run as if they are. Non-git fallback: when no `.git` exists up the tree, the command uses the provided directory as-is.
- **Safe defaults, opt-in for escalation.** Manual over auto, commit over push — the default should always be the safer, more local action.
- **Three command layers, three flag patterns.**
  - **Hook commands** (guard, preserve, protect, pin) — called by Claude Code automatically. Act immediately; no preview needed.
  - **Skill-managed commands** (ship) — `/ship` handles the preview/confirm cycle for `pk changelog` and `pk release`. `--dry-run` exists for the skill to preview before executing. Power users typing `pk changelog` or `pk release` in the terminal bypass the skill and execute directly.
  - **User-only commands** (teardown) — no skill wrapping, destructive. Preview by default, `--confirm` to execute.

### Code Patterns

- **Dependency injection via Config structs.** Every package exports a `Config` struct with injectable deps (`Stdin`, `Stdout`, `Stderr`, `GitExec`, `ReadFile`, etc.) and a `DefaultConfig()` factory wired to real implementations. DI extends to standalone utility functions too: any function that does file I/O accepts injected `readFile`/`writeFile` parameters rather than calling `os.ReadFile`/`os.WriteFile` directly. The call site in `cmd/pk/main.go` passes the real implementations.
- **Tests use Config mocks** — no external test frameworks, no mocking libraries. Tests inject functions that return canned data. Tests use `t.TempDir()` for filesystem tests. Test error paths, not just happy paths: file I/O failures, git operation failures, and config parse errors all need coverage because they protect against silent data corruption.
- **Hook commands** read JSON from stdin, write JSON to stdout, and always exit 0. Shared types and helpers live in `internal/hooks`: `ResolveProjectDir` for project-dir resolution (env var then CWD fallback), `ReadInput` for payload parsing, `WritePostToolUse`/`WritePermissionDecision` for response writing. Response writers return errors; callers log to stderr and continue (hooks never fail on write errors).
- **Shared git helpers** live in `internal/git`: `RepoRoot` (stat-based, no subprocess) resolves a directory to the git repository root and is the single resolution mechanism for all commands. `IsRepo` wraps `RepoRoot` when only the boolean is needed. Commands differ only in failure policy: `changelog` and `release` exit when no repo is found, while `setup` falls back to the given directory (`--allow-non-git`).
- **Managed files** embed a SHA marker (HTML comment for CLAUDE.md, YAML frontmatter `pk_sha256` for skills) so `pk setup` can detect user modifications.
- **Embedded assets** via `//go:embed` — templates, skills, and rules are compiled into the binary.

### Updating pk-managed files

When editing a file that has `pk_sha256` in its frontmatter (skills, rules), update both the embedded source in `internal/setup/` and the local copy in `.claude/`, then recompute the body hash with:

```bash
sed -n '/^---$/,/^---$/!p' <embedded-source> | shasum -a 256
```

Replace the `pk_sha256` line in the local copy with the new value. The sed pattern excludes the frontmatter `---`...`---` block, matching Go's body hash calculation byte-for-byte. This avoids running `pk setup`, which would also touch other managed files.

### Configuration

- `.pk.json` is the project-level config file. Top-level keys map to `pk` subcommands (`changelog`, `guard`, `release`).
- `changelog.types` controls commit type → changelog section mapping.
- `changelog.hooks` supports `preCommit`, `postVersion` lifecycle hooks.
- `release.hooks` supports `preRelease` lifecycle hook.
- `guard.branches` lists branches where git mutations are blocked.
- `release.branch` configures which branch `pk release` merges to and pushes from.

### Documentation

- Convention format: bold principle, then concise context — plain statement when the rule speaks for itself.
- Documentation tight loop: code → tests → command doc (`docs/pk-<command>.md`) → reference docs. New command docs follow `docs/command-doc-template.md`. Reference docs (`docs/pk-json.md`, `docs/error-reference.md`, `docs/environment-variables.md`) centralize information that spans multiple commands: when a change adds a config key, error message, or environment variable, update the relevant reference doc in the same pass. Higher-level docs (README, methodology) link to command docs and only change when concepts change. When they already enumerate options or modes, a new option is a concept change — update them too.
- Terminology: "developer" for the role (reviewing, testing, directing), "builder" for the audience (who plankit serves generally).

### Commits and Releases

- GitHub Actions are pinned to commit SHAs, not mutable tags.

### Hook Protocol

Claude Code hooks receive JSON on stdin and produce JSON on stdout:

- **PreToolUse**: Output `{"decision":"block","reason":"..."}` + exit 0 to block. Exit 0 with no output to allow. Exit 2 blocks the tool call with stderr fed back to Claude — a crashed hook binary fails closed, because Go fatal errors exit 2. Any other non-zero exit (including command-not-found 127) is non-blocking.
- **PostToolUse**: Output `{"systemMessage":"..."}` + exit 0 for user-visible feedback. Use `{"hookSpecificOutput":{"additionalContext":"..."}}` to inject context into Claude's next turn. Both fields can be combined in one response. PostToolUse cannot block (the tool already ran); exit 2 shows stderr to Claude, any other non-zero exit is a non-blocking error.
- **SessionStart**: `.claude/install-pk.sh` bootstraps `pk` into cloud sandboxes. No action needed — if `pk` is on PATH, the script exits immediately.

---
> Source: [markwharton/plankit](https://github.com/markwharton/plankit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
