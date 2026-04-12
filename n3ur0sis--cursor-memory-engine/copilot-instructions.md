## cursor-memory-engine

> Recall shared and local memory before acting


# Memory Recall

- Shared repo memory lives in `.cursor-memory/shared/*.jsonl`.
- Local private memory lives in `.cursor-memory/local/*.jsonl`.
- Before non-trivial work, run `tools/agent/recall.sh` with the best available `--files`, `--tags`, or `--query`.
- Prefer file-scoped recall over broad recall to keep token usage low.
- Treat shared decisions, conventions, workflows, and pitfalls as repo defaults unless the user overrides them.
- Treat local preferences and active session packets as user guidance, not repo-wide truth.
- Ignore expired, archived, or superseded memory entries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/N3ur0sis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/N3ur0sis)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
