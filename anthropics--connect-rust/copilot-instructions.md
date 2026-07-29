## connect-rust

> Before producing a commit, run **both** review agents in parallel (single

# connect-rust — Claude Code Instructions

@CONTRIBUTING.md

## Pre-Commit Code Review

Before producing a commit, run **both** review agents in parallel (single
message, two Agent tool calls):

- `rust-code-reviewer` — correctness, safety, ownership/lifetimes, performance
- `rust-api-ergonomics-reviewer` — downstream-consumer perspective:
  happy-path friction, lints that fire in user crates, runtime footguns,
  doc drift, generated-code readability

The two are complementary (different lenses on the same diff) and produce
largely non-overlapping findings. Address all **Critical**, **High**, and
**Medium** findings from both. For **Low** / advisory findings, flag these
to the user to decide.

For changes that touch only internal/test/bench code with no public-API
surface, the ergonomics reviewer may be skipped.

---
> Source: [anthropics/connect-rust](https://github.com/anthropics/connect-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
