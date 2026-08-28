## korg-bro

> Operational 2-minute map: [CHEATSHEET.md](CHEATSHEET.md).

# Korg architecture

Operational 2-minute map: [CHEATSHEET.md](CHEATSHEET.md).

Korg is a React + Tauri desktop client for Grok Build. The React UI connects through a Tauri host to the current Grok ACP.

## Ownership

```
React UI -> Tauri host -> one `grok agent --leader stdio` proxy -> Grok leader
                         |
                         +-> `updates.jsonl` transcript projection
                         +-> `~/.grok/korg` UI overlay
```

- Grok ACP is the source of truth for sessions, queueing, auth, billing, skills, hooks, plugins, MCP, trust, and Grok data mutations.
- Grok-facing actions are sent through ACP.
- `updates.jsonl` supplies transcript hydration and history detail.
- `~/.grok/korg` stores Korg presentation state such as aliases, pins, read markers, theme, and recent projects.

## Code boundaries

| Concern | Location |
| --- | --- |
| ACP process, RPC, load, prompt, cancel | `apps/korg/src-tauri/src/acp.rs`, `agent_link.rs`, `agent_host.rs` |
| Current ACP wire parsing | `apps/korg/src-tauri/src/acp_msg.rs` |
| Journal transcript replay/detail | `apps/korg/src-tauri/src/session_index/` |
| Korg-owned UI overlay | `apps/korg/src-tauri/src/overlay.rs` |
| Session selection, hydrate, live projection | `apps/korg/src/session/` |
| UI ACP types | `apps/korg/src/acpTypes.ts` |

## Invariants

1. The current ACP method and field names define the wire contract.
2. One leader proxy and one backend mutation lane serialize Grok mutations. Async operations keep `AppState` access and stdin writes within brief synchronous sections.
3. Logout is ordered: close loaded sessions, call `x.ai/auth/logout`, then drop the proxy. A failed step returns an error and leaves the proxy usable.
4. Event ids are opaque strings and are compared as complete values for equality.
5. Session selection has one revision counter. Hydration commits against the captured revision while it remains current.
6. Each follow-up has one submission. Grok's `x.ai/queue/*` notifications provide its authoritative status after acceptance.
7. Hydration reconciles journal data with live events that arrived during the read. Live data wins over an older snapshot.
8. `x.ai/sessions/list` is the source of truth for roster data.
9. Korg overlay writes are atomic and serialized under `~/.grok/korg`.
10. New Tauri commands require the command handler and permission entry. ACP view changes require matching Rust and TypeScript types.

## Implementation shape

- Put pure session rules in `src/session/*.ts` with colocated tests; keep `App.tsx` as wiring.
- ACP results drive Grok-facing state, the session store holds the live UI projection, and overlay transactions persist presentation preferences.
- Performance work follows measured hot paths. Chat items subscribe separately from app chrome, and asynchronous polling is single-flight.

## Checks

```sh
pnpm test
pnpm lint
pnpm build
pnpm exec tsc --noEmit -p apps/korg/tsconfig.json

cd apps/korg/src-tauri
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test
```

---
> Source: [lynnzc/korg-bro](https://github.com/lynnzc/korg-bro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
