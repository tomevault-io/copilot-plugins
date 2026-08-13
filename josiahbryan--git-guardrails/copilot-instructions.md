## git-guardrails

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

git-guardrails is a compiled Bun/TypeScript binary that shadows `/usr/bin/git` via PATH priority. It intercepts every git invocation system-wide, blocks dangerous commands (especially from AI coding agents), and transparently passes safe commands through to the real git at `/usr/bin/git`.

## Commands

```bash
bun test              # Run all tests (Bun's built-in test runner)
bun build --compile --outfile dist/git src/index.ts  # Build native binary
bun run install-guardrails   # Build + install to /opt/homebrew/bin/git
bun run release              # Cross-compile for 4 targets + create GitHub release
```

During development, run the wrapper without compiling:
```bash
bun run src/index.ts status   # Test a safe command
bun run src/index.ts stash    # Test a blocked command
```

## Architecture

Four source files, no external dependencies:

- **`src/rules.ts`** — `DANGEROUS_RULES` array of `DangerousRule` objects. Each rule has a `subcommand` string, a `match(args)` predicate, and a `reason` string. To add a new blocked command, add an entry here.
- **`src/matcher.ts`** — `checkCommand(args)` iterates rules, returns `{ blocked, reason }`.
- **`src/plugin.ts`** — Identity-injection plugin contract (see "Plugin contract" section below). Invoked AFTER the block check, only for commit-creating subcommands, and only when `GIT_GUARDRAILS_IDENTITY_PLUGIN` is configured.
- **`src/index.ts`** — Entry point. Parses argv (handles bun-run vs compiled-binary differences), checks `GIT_ALLOW_DANGEROUS` env var, calls `checkCommand`, either blocks with exit 128 or runs the plugin (if configured + commit-creating) then `execFileSync`s the real git.

## Plugin contract (identity injection)

Set `GIT_GUARDRAILS_IDENTITY_PLUGIN=/path/to/executable` to inject extra identity / provenance into every commit-creating git invocation. The plugin is invoked with no positional args, no stdin, a 3-second timeout, and receives the about-to-run subcommand via `GIT_GUARDRAILS_SUBCOMMAND` in env.

The plugin emits `KEY=VALUE` lines on stdout (uppercase keys only — `^[A-Z_][A-Z0-9_]*$`). Two recognized shapes:

- **`COMMIT_TRAILER=<full trailer line>`** — wrapper translates each one into a `--trailer "<value>"` arg in the git argv. Multiple `COMMIT_TRAILER=` lines produce multiple `--trailer` args. When any trailer is emitted, `-c trailer.ifExists=addIfDifferentNeighbor` is prepended (top-level git flag) so rebase replays dedupe duplicates via git's own mechanism — no state tracking in the wrapper.
- **Anything else (`GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL`, `GIT_AUTHOR_*`, or a custom key)** — exported into the env passed to the real-git exec.

Plugin failures (non-zero exit, timeout, unreadable executable) log a one-line `[git-guardrails] identity plugin failed (<path>): <msg>` to stderr and the wrapper proceeds with no injection. A broken plugin must never break git.

The plugin only runs on commit-creating verbs (`commit`, `cherry-pick`, `revert`, `merge`, `rebase`); everything else (`status`, `log`, `diff`, `fetch`, `push`, …) pays zero plugin overhead.

## Key Design Decisions

- **Exit code 128** signals fatal git error to discourage agent retries.
- **`GIT_ALLOW_DANGEROUS=1`** bypasses all checks. Intentionally never mentioned in error output so agents don't discover and use it.
- **`gitArgvFromProcessArgv()`** handles three execution contexts: `bun run src/index.ts`, compiled binary (Bun `/$bunfs/` path), and direct invocation.
- **`git restore` is blocked entirely** (not just `restore .`), because path-specific restores were bypassing the previous narrow rule.

## Tests

Tests live in `tests/` using `bun:test`:

- **`rules.test.ts`** — Validates rule structure.
- **`matcher.test.ts`** — Unit tests for blocked/allowed/edge-case commands (~44 cases).
- **`integration.test.ts`** — Subprocess tests running actual `bun run src/index.ts` and the compiled `dist/git` binary, including `GIT_ALLOW_DANGEROUS` bypass.

Integration tests depend on the compiled binary existing at `dist/git` — run `bun run build` first if those tests fail.

---
> Source: [josiahbryan/git-guardrails](https://github.com/josiahbryan/git-guardrails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
