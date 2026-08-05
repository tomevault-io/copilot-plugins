## vtcode

> [Root AGENTS.md](../../../AGENTS.md) | Per-session event log and derived state.

# vtcode-memory

[Root AGENTS.md](../../../AGENTS.md) | Per-session event log and derived state.

## Conventions

- `events.jsonl` is canonical; `derived/` and `index/` are regenerated views — never persist session history elsewhere.
- `progress.rs` hosts the `GoalTracker` state machine and compaction-safe `ProgressLedger` view.
- Append-only: do not mutate historical events; new facts go through `append`.
- Off the hot path: never read the log back into agent context; use derived queries for revert/compaction/analytics.
- Public API uses `anyhow::Result<T>` + `.context()`; no `unwrap`/`expect` in non-test code.
- Keep ordinary appends buffered; flush at turn boundaries, reads, cap rewrites, and close.
- Return persistence errors to callers; do not silently discard cap-enforcement failures.

## Dependencies

- `vtcode-exec-events` owns the `ThreadEvent` / `VersionedThreadEvent` contract; never reinvent event types.
- `walkdir` handles directory-size and GC walks; `chrono`, `serde`, and `serde_json` handle persistence metadata.
- `uuid` supports verifier-id generation for the goal tracker.

## Testing

- Use `cargo nextest run -p vtcode-memory` and cover ordering, reopen/index reconstruction, retention, and write boundaries.

---
> Source: [vinhnx/vtcode](https://github.com/vinhnx/vtcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
