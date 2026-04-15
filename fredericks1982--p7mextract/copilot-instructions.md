## p7mextract

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

p7mextract is a Bash CLI tool for extracting content from digitally signed `.p7m` files (PKCS#7 format) on macOS. It uses OpenSSL's smime capability, auto-detecting between DER and PEM encoding.

## Commands

```bash
make test                        # Run all tests (default target, generates fixtures first)
make test-verbose                # Run tests with TAP output
make test-filter FILTER="pattern" # Run tests matching a name pattern
make clean                       # Remove generated test fixtures
make install                     # Install to ~/.local/bin
```

Testing uses **Bats** (Bash Automated Testing System) with bats-assert, bats-support, and bats-file helpers. Bats libraries are expected at `/opt/homebrew/lib/` (Apple Silicon) or `/usr/local/lib/` (Intel).

## Architecture

Single executable script (`p7mextract`, ~300 lines) structured as:

1. **Color/output helpers** — `msg_success()`, `msg_error()`, `msg_warning()`, `msg_prompt()` with Unicode icons
2. **Utility functions** — `get_default_output()` (smart filename: `file.pdf.p7m` → `file.pdf`), `check_dependencies()`, `confirm_overwrite()`, `get_human_size()`
3. **Core extraction** — `extract_p7m()` tries DER first, falls back to PEM, cleans up on failure
4. **Main function** — argument parsing (`-o`/`--output`, `-h`/`--help`), validation, interactive prompts with defaults

## Test Structure

- `test/p7mextract.bats` — 40+ test cases covering extraction, filename logic, error handling, interactive mode, edge cases
- `test/test_helper.bash` — shared setup (temp dirs, fixture paths, helper functions)
- `test/fixtures/setup_fixtures.sh` — generates self-signed certs and signed sample files; auto-run by `make test`

## Git Workflow

- **Never push directly to `master`.** Always create a feature branch and open a pull request.
- The `master` branch has protection rules requiring PRs and passing status checks.

## Platform Notes

- macOS-specific: uses `stat -f%z` for file size and `bc` for formatting
- Requires OpenSSL with smime support

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fredericks1982)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fredericks1982)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
