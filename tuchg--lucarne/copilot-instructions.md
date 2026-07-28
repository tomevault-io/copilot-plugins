## lucarne

> - Raw agent session APIs and the shared semantic layer are different layers. Do not mix them.

# agent-sessions Development Rules

## Core Boundary
- Raw agent session APIs and the shared semantic layer are different layers. Do not mix them.
- `agent_sessions::providers::<agent>::...` (re-exported at the crate root as `agent_sessions::<agent>`) is for agent-specific typed schema.
- `agent_sessions::agent_session::...` is the shared semantic layer (`Session`, `Event`, `Body`, …).
- Do not make raw agent types depend on shared `agent_session` types just to reduce duplication.
- If a stable raw field already carries shell semantics, parse that shell subset in the raw agent API. Do not make downstream crates reopen agent-specific JSON blobs just to recover `command`, `duration`, or shell/non-shell classification.
- Each agent implements `agent_session::IntoAgentSession` in its own module. Do not rebuild a central cross-agent mapper full of provider-specific `match` arms.

## Unknown Handling
- `Unknown` is a compatibility fallback, not a normal endpoint.
- Before inventing a new public variant name, scan real local session inventory first. Use observed key sets and subtype counts to confirm the shape is actually stable.
- If a real session sample, vendored fixture, or benchmark sample contains an `Unknown` kind, promote it to a typed variant as part of the same change whenever the shape is stable enough to model.
- Do not demote an unrecognized nested subtype to an entry-level `Unknown` if the enclosing family is already known. Keep the fallback inside that family and preserve the family context.
- Every promoted `Unknown` must add or update:
  - parser coverage
  - a fixture or inline regression test
  - event mapping when `event` is enabled
- Leaving a new `Unknown` in place is only acceptable when the shape is genuinely unstable or still unclear. In that case, preserve the raw payload and add a note in the PR/task summary explaining why it stayed unknown.

## Schema Fidelity
- Do not trust field names inferred from Rust naming conventions or `rename_all` alone. Verify critical payload keys against real session samples.
- Do not trust field types either. Session schemas drift between string, number, object, and null in the same logical field. Parse the real observed shapes and keep the public type honest.
- When promoting a shape into a typed variant, confirm the raw field mapping for every payload branch that populates the new public fields.
- If a test only proves the outer event kind, tighten it until it asserts the parsed payload fields too. A typed variant with silently empty fields is a parser bug, not a success.

## Feature Boundaries
- Features control surfaces, not random helper branches.
- Avoid `#[cfg(feature = ...)]` inside generic helper code unless the helper truly depends on an optional dependency.
- Prefer this structure:
  - agent-local parsing in `src/providers/<agent>/...`
  - agent-local discovery in `src/providers/<agent>/discovery.rs`
  - agent-local semantic mapping in `src/providers/<agent>/event.rs`
- If adding a new agent-specific schema helper, put it under that agent module first. Only move it to a shared module after at least two agents need the exact same behavior.
- Do not turn `src/util.rs` into a parser helper dumping ground. Keep generic string/path primitives there; keep JSON shape helpers in the owning agent module.

## File Layout
- Keep each supported agent under its own module directory in `src/providers/<agent>/`.
- Prefer splitting by responsibility:
  - `types.rs` for public raw types
  - `mod.rs` or `parse.rs` for parsing logic
  - `discovery.rs` for filesystem discovery
  - `event.rs` for the agent-specific `IntoAgentSession` implementation
- Keep `src/agent_session/mod.rs` focused on shared semantic types and trait definitions. Agent-specific projections belong in `src/providers/<agent>/event.rs`.

## Performance Rules
- Use `serde` and `serde_json::Deserializer` for parsing. Do not hand-roll JSON tokenization.
- Avoid `serde_json::Value` on hot paths unless there is no stable typed alternative.
- Borrow first, allocate later.
- For immutable parsed result trees, prefer `Box<[T]>` over `Vec<T>` once construction is complete.
- Before keeping a performance optimization, benchmark it against the simpler `serde` path.
- Remove negative-value optimizations and dead compatibility code once benchmarks show they are not helping.

## Verification
- Run `cargo test -p agent-sessions --all-features` after structural changes.
- Run at least the relevant bench compile check after parser refactors:
  - `cargo bench -p agent-sessions --features <agent> --bench <bench> --no-run`
- If a refactor changes parsing behavior for a real local sample, mention that sample shape in the task summary.

---
> Source: [tuchg/Lucarne](https://github.com/tuchg/Lucarne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
