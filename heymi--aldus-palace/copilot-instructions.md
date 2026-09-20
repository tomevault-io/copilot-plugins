## aldus

> Aldus Palace project rules


- `packages/core` is runtime-agnostic: it never imports Hono, Express or Workers
  APIs and never reads `process.env`.
- Everything that touches storage is `async`; the storage port is the only I/O
  boundary.
- Every mutation writes an `action_log` entry via `writeActionLog`.
- Never edit a shipped migration; add a new one.
- Tests are deterministic and offline. Add or extend a suite under
  `packages/core/test` for logic changes, and an `eval/fixtures` entry for
  user-visible understanding changes.
- Docs describe shipped behaviour. Planned work belongs in `ROADMAP.md` and
  `docs/INTELLIGENCE.md` under a `Designed` marker.
- Run `pnpm verify` before proposing a change.

The full contract is [`AGENTS.md`](../AGENTS.md); every claim maps to a proof in
[`EVALUATION.md`](../EVALUATION.md).

---
> Source: [heymi/aldus-palace](https://github.com/heymi/aldus-palace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
