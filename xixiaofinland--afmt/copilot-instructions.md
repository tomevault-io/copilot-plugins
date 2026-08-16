## afmt

> `afmt` is a Rust CLI and library for formatting Salesforce Apex. Keep this file

# Agent guidance

`afmt` is a Rust CLI and library for formatting Salesforce Apex. Keep this file
as a navigation aid for coding agents; put durable behavior details in the
focused project documentation below.

- Formatter configuration and style behavior:
  [docs/configuration.md](docs/configuration.md)
- Comment placement and formatter idempotency:
  [docs/comment-placement-and-idempotency.md](docs/comment-placement-and-idempotency.md)
- User-facing bulk formatting, configuration, discovery, and CLI semantics:
  [docs/project-wide-formatting.md](docs/project-wide-formatting.md)
- Test, battle-corpus, and local benchmark workflow:
  [docs/validation-and-benchmarks.md](docs/validation-and-benchmarks.md)
- Basic installation and usage examples: [README.md](README.md)

When changing formatting behavior, update the focused guide and its relevant
fixtures/tests together. Keep `CLAUDE.md` unchanged.

---
> Source: [xixiaofinland/afmt](https://github.com/xixiaofinland/afmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
