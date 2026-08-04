## tar-diff

> This file provides guidance to AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

`tar-diff` is a Go library and CLI toolset for diffing and patching tar files, primarily used for efficient distribution of OCI container images.

## Agent Workflow Guidelines

- **For simple changes**: Propose a short plan and ask a human to review it before you start implementing it.
- **Never assume or guess** intent or meaning. If instructions, code, or documentation are ambiguous, ask for clarification.
- **If you find a potential bug or security issue** in the code, inform the developer.
- **Follow development conventions**: All code style, commit formatting, and testing requirements are in [`CONTRIBUTING.md`](CONTRIBUTING.md).

## AI Permissions

- **Always allowed**: Read any file, run linting commands (`make validate`, `golangci-lint`), search GitHub issues and PRs, analyze logs and test output.
- **Safe to change**: Log messages, comments, variable names, code formatting.
- **Ask first**: Anything that affects core algorithms, file format, or cross-platform compatibility. This includes: documentation files (`README.md`, `CONTRIBUTING.md`, `AGENTS.md`), test assertions, linting config (`.golangci.yml`), refactoring, binary diff and patch logic (`pkg/tar-diff/`, `pkg/tar-patch/`), file format specification (`file-format.md`), CLI interface changes (`cmd/`), build configuration (`Makefile`, `.goreleaser.yml`), CI workflows (`.github/workflows/`), dependency changes (`go.mod`), platform-specific code (`cmd/tar-diff/tmpdir_*.go`).
- **Never**: Push directly to `main`, commit compiled binaries (`tar-diff`, `tar-patch`), break cross-platform compatibility, modify the tardiff file format without approval, log or hardcode sensitive information.

## Platform Requirements

Code must work on Linux (x86_64, ARM), macOS (ARM), and Windows (x86_64).

## File Format

For tardiff file format work, see [`file-format.md`](file-format.md) for the complete binary format specification.

---
> Source: [containers/tar-diff](https://github.com/containers/tar-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
