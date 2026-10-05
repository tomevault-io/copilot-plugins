## jiti

> This project is a cooperative live SBCL image-repair kernel. Preserve active restart dynamic extent, worker ownership, managed code/data rollback, and the distinction between goal predicates and safety checks.

# Repository instructions

This project is a cooperative live SBCL image-repair kernel. Preserve active restart dynamic extent, worker ownership, managed code/data rollback, and the distinction between goal predicates and safety checks.

When work introduces, changes, or supersedes an architectural decision, use **$maintain-adrs** and read [its instructions](skills/maintain-adrs/SKILL.md). Keep [ADRs](docs/adr/) consistent with implementation and verification. Each ADR captures one decision and its rationale; supersede changed decisions instead of rewriting historical choices.

Run `devenv shell test` for kernel changes and `python3 scripts/check-adrs.py` for ADR changes. Run `devenv shell test-live` when modifying the Responses integration and local credentials are configured. Property failures must produce a replayable minimized trace. Do not put credentials in source, events, failure artifacts, or ADRs.

---
> Source: [ghuntley/jiti](https://github.com/ghuntley/jiti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
