## daw-ai

> This file tells coding agents how to work productively in this repository.

# Agent instructions for daw-ai

This file tells coding agents how to work productively in this repository.

## charter.md

Treat the charter.md file as the authoritative project document from the user. NEVER modify it yourself,
unless the user explicitly instructs you to. It should only be modified by the user. Your job is
to implement the project and features that they describe in this document.

## Before opening a PR

**Run `just test` and confirm it passes before opening a PR.**
This target runs the `pre` recipe first, which checks Rust formatting, runs Clippy with warnings
denied, and checks the JavaScript syntax. It then runs the Rust test suite, builds the server,
and exercises the core UI workflows in headless Chrome or Chromium.
If any of those fail, fix the underlying issue; do not bypass checks.

## Style guide

- Comments should be brief and focus on important invariants, architectural details, or other
  long-term relevant information. They should not contain minor implementation details of the current
  commit.
- DO NOT use non-ASCII characters in the codebase. Escaped characters are acceptable, but not raw characters

## Tests

When adding new features, add tests, but aim for high code coverage and important integration
tests without adding too many lines of new test code. Expanding a logically related existing test is
often a good way to achieve coverage without bloating the suite.

## Git commits

1. Make one commit per feature or bug fix when opening a PR. Multiple commits or fixup commits should
   not be merged to the default branch.

---
> Source: [cberner-ai-org/daw-ai](https://github.com/cberner-ai-org/daw-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
