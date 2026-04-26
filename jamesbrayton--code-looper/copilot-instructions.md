## code-looper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code Looper is a Rust CLI application that provides a pluggable loop engine for multiple coding-agent CLIs (Claude Code, GitHub Copilot CLI, Codex CLI). It supports configurable iteration behavior, policy-driven orchestration flows, and structured logging. See `docs/PRD.md` for full requirements.

## Architecture

The application is organized into these layers (per PRD):
- **CLI Layer**: Argument parsing, config validation, runtime initialization
- **Loop Engine**: Iteration scheduler, stop conditions, retry/backoff, completion hooks
- **Provider Adapter Layer**: Common trait for executing prompts across Claude Code CLI, GitHub Copilot CLI, and Codex CLI
- **Orchestration Policy Engine**: Evaluates repo context (PRs, issues), selects workflow branch, generates provider prompt payloads
- **Policy Guard Layer**: Enforces MCP-only GitHub mutation path
- **Workspace Bootstrap/Validation**: Checks/initializes repo prerequisites before loop execution
- **Context Integrations**: GitHub MCP-backed resolver for PR/issue state
- **Telemetry/Logs**: Structured per-iteration logs and session summaries

## Build & Run Commands

```bash
cargo build              # Build debug
cargo build --release    # Build release
cargo run                # Run debug build
cargo test               # Run all tests
cargo test <test_name>   # Run a single test
cargo clippy             # Lint
cargo fmt                # Format code
cargo fmt -- --check     # Check formatting without modifying
```

## Development Environment

- **Dev container**: Rust image (`mcr.microsoft.com/devcontainers/rust:2-1-trixie`) with Node LTS, GitHub CLI, Claude Code, Copilot CLI, and uv pre-installed
- **Environment variables**: Stored in `.env` (gitignored). Requires `GITHUB_TOKEN` for MCP GitHub integration.

## MCP Servers

Configured in `.mcp.json`:
- **context7**: Library documentation lookup
- **markitdown**: Document format conversion
- **microsoftdocs**: Microsoft Learn / Azure documentation
- **github**: GitHub API (uses `GITHUB_TOKEN` from `.env`)

## Workflow Requirements

**Every change must be tracked by a GitHub issue.** No commits without an associated issue.

- Before starting work, check for an existing GitHub issue using the `github-issues` skill or MCP GitHub tools
- If no issue exists, create one before making changes
- Reference the issue number in all commit messages (e.g., `Fix data ingestion reconnect logic (#12)`)
- Issue comments are the canonical execution log while work is in progress.
  - Add comments at meaningful milestones, such as:
    - Scope clarified and implementation plan finalized
    - First implementation pass complete
    - Tests added/updated and results captured
    - Blocker discovered or dependency identified
    - Handoff to another agent/person
  - In the same update cycle, keep the issue body current:
    - Update checklist progress
    - Update notes/decisions sections
    - Add or revise blockers/dependencies
  - Ensure handoff quality:
    - A new contributor should be able to read the issue and resume work immediately
    - Include what changed, what is left, and where to continue
    - Reference repo state when relevant (branch, PR, commit, failing test, artifact)
- Ensure issues are updated and closed when work is complete

## Key Documentation

- `docs/PRD.md` — Full product requirements document with architecture, milestones, risks, and success criteria
- `docs/ADRs/` — ADRs tracking significant architectural decisions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesbrayton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
