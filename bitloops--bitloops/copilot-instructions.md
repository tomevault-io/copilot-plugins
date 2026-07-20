## bitloops

> Never run all tests unless asked to. Prefer running relevant tests, to the change being made.

Never run all tests unless asked to. Prefer running relevant tests, to the change being made.
Do not use git worktrees when there is no need to parallelise, or not specifically asked to. Prefer subagents
Prefer cargo nextest-backed repo aliases over cargo test; use cargo test only for documented exceptions.
For daemon, supervisor, background worker, server/API, and other long-lived backend flows, use the shared main logger and follow documentation/contributors/guides/daemon-service-logging.md. Do not introduce silent terminal failures.
If you run any qat tests, prefer to run them with --parallel flag

---
> Source: [bitloops/bitloops](https://github.com/bitloops/bitloops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
