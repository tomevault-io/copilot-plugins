## ai-tools

> This repository contains tools for Claude Code to operate more efficiently. Each tool is a separate module under its own directory.

# ai-tools Development Guidelines

## Project Overview

This repository contains tools for Claude Code to operate more efficiently. Each tool is a separate module under its own directory.
Cross-tool shared code lives under `shared/` and should stay domain-neutral instead of absorbing tool-specific workflows.

## Release Policy

### Binary Distribution

All tools with compiled binaries MUST provide pre-built binaries for users without build tools installed.

**Supported Platforms:**
- `darwin-arm64` (macOS Apple Silicon)
- `darwin-amd64` (macOS Intel)
- `linux-amd64` (Linux x86_64)
- `windows-amd64` (Windows x86_64)

### Version Management

- Tag format: `v<MAJOR>.<MINOR>.<PATCH>` (e.g., `v1.2.0`)
- All tools share the same version and are released together
- `ensure-binary.sh` and `install.sh` fetch the latest version from GitHub API
- Plugin `version` fields are auto-updated by CI on release

### Release Process

1. Commit changes
2. Create and push tag: `git tag v1.x.x && git push --tags`
3. `release.yml` orchestrates:
   - N product workflows build binaries and attach to GitHub Release
   - `cc-marketplace-release.yml` updates `plugin.json` versions + regenerates `marketplace.json` + auto-commits to main

## CI/CD

### Workflow Structure

```
.github/workflows/
├── release.yml                  # Orchestrator (tag v* trigger)
├── <tool>-release.yml           # Product build (reusable, called by release.yml)
├── cc-marketplace-release.yml   # Version + marketplace sync (reusable, called by release.yml)
├── <tool>-test.yml              # Test workflow (push to main + PRs, path-filtered)
└── .github/scripts/
    └── sync-marketplace.sh      # Generates marketplace.json from plugin.json files
```

**Orchestrator** (`release.yml`):
- Triggers on tag push matching `v*`
- Calls all `<tool>-release.yml` workflows in parallel
- Then calls `cc-marketplace-release.yml` to sync versions

**Product release** (`<tool>-release.yml`):
- Triggered via `workflow_call` with `version` input
- Builds cross-platform binaries and attaches to GitHub Release

**Marketplace sync** (`cc-marketplace-release.yml`):
- Triggered via `workflow_call` with `version` input
- Updates `version` in each `plugin.json`, regenerates `marketplace.json`, commits to main

**Test workflow** (`<tool>-test.yml`):
- Triggers on push to `main` and PRs with `paths: ['<tool>/**']`
- Runs `go test ./...`

### Adding New Workflows

When adding a new tool:
1. Create `<tool>-test.yml` with path filter for `<tool>/**`
2. Create `<tool>-release.yml` as reusable workflow (`on: workflow_call`)
3. Add the tool to `release.yml` orchestrator

## Plugin Structure

Each tool should follow this structure:

```
<tool-name>/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest
├── install.sh            # One-liner install script (curl | bash)
├── Makefile              # Build workflow (build, cross, test, clean)
├── skills/               # Claude Code skills
├── skills-codex/         # OpenAI Codex skills
├── scripts/
│   └── ensure-binary.sh  # Auto-download script
├── cmd/
│   └── <tool>/           # CLI entry point
├── internal/             # Shared logic
└── dist/                 # Built binaries (gitignored)
```

### Adding New Tools

1. Create tool directory with the standard plugin structure above
2. Create `.github/workflows/<tool>-test.yml` and `<tool>-release.yml` (reusable)
3. Add the tool to `.github/workflows/release.yml` orchestrator
4. Include `category` field in `.claude-plugin/plugin.json`
5. Update root `README.md` with the new tool

## Skills

Claude Code skills go in `skills/` under each tool directory. Codex-specific skills go in `skills-codex/`. The two may share the same skill names but differ in prompt content:

- `skills/` — Claude Code skills. Use `/skill-name` invocation syntax and Claude Code features (e.g., `EnterPlanMode` tool, `/loop`).
- `skills-codex/` — OpenAI Codex skills. Use `$skill-name` invocation syntax. Prompts must avoid Claude Code-specific features and default backend to `codex`.

When adding or updating a skill, check if it exists in both directories and keep them in sync where the logic is shared.

## AGENTS.md Symlink Convention

Every `CLAUDE.md` in this repo must have a corresponding `AGENTS.md` symlink (`ln -s CLAUDE.md AGENTS.md`) in the same directory. When creating or moving a `CLAUDE.md`, always create the symlink alongside it.

## CLI Documentation Philosophy

These tools are used by AI agents. Documentation should optimize for discoverability, not exhaustiveness.

Principles:
- Every CLI must be fully discoverable via `<tool> --help` and `<tool> <subcommand> --help`
- Do NOT duplicate help output into documentation — it goes stale
- If a tool has a schema/reference command (e.g., `webform schema`), point to it instead of inlining the reference

AI-facing docs (CLAUDE.md, skill prompts) should contain only:
- One-line description: what the tool does and when to use it
- Non-obvious rules: workflow patterns, conventions, and constraints that `--help` doesn't convey
- One concrete example showing the core workflow
- Discovery pointers: which help/schema commands to run for details

## Naming: EDR-Safe Conventions

Go module and package paths are embedded in Go binaries through build info, symbol names, and file paths. EDR tools may use these strings as part of heuristic analysis.

Avoid EDR-sensitive names in package names, module paths, and directory names, especially for binaries that also use daemonization, self re-exec, process inspection, or IPC.

Names to avoid in this repo:
- `agent`, `agentbus`, `agentirc`
- `payload`, `beacon`, `implant`, `dropper`, `loader`, `injector`, `shellcode`

This project includes binaries that use daemon self-reexec (`os.Executable` + `exec.Command`), process tree walking (`ps`), and Unix socket IPC. Combined with aggressive naming, those patterns can trigger false positives.

## Code Style

- **Language**: Go for performance-critical tools
- **Error Handling**: Always return meaningful error messages
- **Documentation**: Include CLAUDE.md in each tool directory with usage instructions

---
> Source: [bang9/ai-tools](https://github.com/bang9/ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
