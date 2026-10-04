## privacy-and-hooks-mdc

> Privacy and hook hot-path constraints; apply when touching events, adapters or hooks


- Hook entrypoint code is **stdlib-only** with lazy imports and must always fail open (exit 0; print `{"permission":"allow"}` for permission hooks, `{}` for the rest).
- Parse hook payloads through an allowlist at the boundary. Never persist prompts, thinking or response text, file contents, command output, env, `user_email` or `transcript_path`.
- Do not register `beforeSubmitPrompt`, `afterAgentThought`, `afterAgentResponse` or `beforeReadFile` in v0.1; keep a test that generated hook config never contains them.
- Hooks only append to per-session JSONL spools; only the indexer writes SQLite.
- Runtime dir is `<git-common-dir>/cursorfleet/` with 0700 dirs and 0600 files.
- No network calls anywhere in the hook path.

---
> Source: [M9nx/cursorfleet](https://github.com/M9nx/cursorfleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
