## redis

> Pure-TypeScript Redis / Valkey wire-protocol driver. Sibling of **Tusk**

# @perry/redis

Pure-TypeScript Redis / Valkey wire-protocol driver. Sibling of **Tusk**
(the GUI that consumes it) and of **@perry/postgres** + **@perry/mysql**
(the other Perry-showcase drivers). Published independently as
`@perry/redis` and usable by any Perry, Node.js, or Bun program that
wants to talk to Redis 7+, Redis 8+, or Valkey 8+.

## Positioning

- **Showcase of Perry's systems-programming capability.** No native crate
  in this package; all capabilities come from perry-stdlib
  (`net.Socket`, `tls.connect`, `crypto.*`, `Buffer`).
- **Runs unchanged on Node.js / Bun.** Redis uses initial TLS only (no
  mid-stream upgrade), so the transport story is even simpler than
  pg/mysql — `tls.connect` from SYN.
- **Shaped for a GUI**, not an ORM. `CommandReply` carries the decoded
  value, the raw RESP type tag, and the post-CRLF bytes for byte-exact
  round-trip. Push frames (RESP3 invalidations, keyspace notifications)
  surface separately from command replies.

## Architecture

```
TypeScript driver
    │
    ├── src/protocol/       RESP2 + RESP3 framing, reader, writer            (R1)
    ├── src/transport/      TCP + TLS-from-SYN socket adapter                (R2, R5)
    ├── src/auth/           HELLO 3 + legacy AUTH dispatcher                 (R3)
    ├── src/connection.ts   lifecycle, FIFO command queue, subscribe state  (R2+)
    ├── src/pipeline.ts     single-write batch + ordered FIFO pop            (R4)
    ├── src/pubsub.ts       channel/pattern callbacks; RESP2+RESP3 paths    (R4)
    ├── src/multi.ts        MULTI/EXEC/DISCARD/WATCH builder                 (R6)
    ├── src/scripts.ts      EVAL/EVALSHA + auto-NOSCRIPT retry               (R6)
    ├── src/tracking.ts     CLIENT TRACKING + invalidation push routing     (R6)
    ├── src/blocking.ts     pool-aware blocking-command wrapper              (R6)
    ├── src/cluster.ts      slot table + MOVED/ASK + CLUSTER SHARDS         (R6)
    ├── src/pool.ts         connection pool with RESET-on-release            (R5)
    ├── src/url.ts          redis:// + rediss:// + unix:// DSN parser       (R5)
    ├── src/env.ts          REDIS_* env vars                                 (R5)
    ├── src/error.ts        RedisError + subclass classification            (R2+)
    ├── src/cancel.ts       CLIENT KILL ID via second connection            (R6)
    └── src/index.ts        public barrel exports
```

## Milestones (mirror the plan at /Users/amlug/.claude/plans/just-like-mysql-and-harmonic-sparkle.md)

- **R1** — RESP2 + RESP3 framing + reader + writer. Unit tests green.
- **R2** — TCP + connect + PING/GET/SET against mock server.
- **R3** — HELLO 3 + RESP2 fallback; legacy + ACL AUTH.
- **R4** — Pipelining + Pub/Sub (RESP2 message + RESP3 push paths).
- **R5** — TLS-from-SYN + Pool + URL/env.
- **R6** — Cluster + MULTI/EXEC + Scripts + Tracking + Cancel + Blocking commands.
- **R7** — Sentinel, benchmarks, polish, docs (deferred session).

## Node.js compatibility contract

Code under `src/` must only use APIs available both in Perry's stdlib and
Node.js core:

- `Buffer` (same API on both), `Buffer.concat`.
- `net.createConnection(host, port)` with `.on('connect'|'data'|'error'|'close')`.
- `tls.connect({host, port, ...})` for `rediss://`.
- `crypto.createHash`, `crypto.randomBytes`, etc. (only used if a custom
  auth plugin needs them — core is crypto-free).

The one divergence is the cross-env `net.createConnection` signature:

- Perry: `net.createConnection(host, port)`
- Node:  `net.createConnection({ host, port })`

`src/transport/net-socket.ts` is the only place in the driver that
cares — feature-detected via `process.versions.node`.

## Perry AOT constraints (apply to all source files)

Per the hone CLAUDE.md conventions:

- No `?.` optional chaining — use explicit `if (x === undefined)` / `if (x === null)`.
- No `??` nullish coalescing — use explicit branching.
- No `obj[variable]` dynamic key access — use `if/else if` or switch.
- No `/regex/.test()` — use `indexOf` or char-code checks.
- No `{ key }` ES6 shorthand — write `{ key: key }`.
- No `for...of` on arrays — use `for (let i = 0; i < arr.length; i++)`.
- No `setTimeout` self-recursion — use `setInterval`.
- No closures capturing instance methods as `this.method` — store state in
  module-level `Map<id, State>` and use named module-level handlers.

## Testing

```bash
bun test                    # all tests
bun test tests/unit         # pure unit tests (no network)
bun test tests/integration  # in-process mock server
docker compose -f tests/integration/docker-compose.yml up -d
REDIS_HOST=127.0.0.1 REDIS_PORT=36379 bun test tests/integration/real-server.test.ts
```

---
> Source: [PerryTS/redis](https://github.com/PerryTS/redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
