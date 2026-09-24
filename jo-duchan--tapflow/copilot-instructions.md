## tapflow

> > Common rules: [AGENTS.md](../../AGENTS.md) | Full index: [INDEX.md](../../INDEX.md)

# test-utils — AGENTS.md

> Common rules: [AGENTS.md](../../AGENTS.md) | Full index: [INDEX.md](../../INDEX.md)

---

## WHAT

Test-only helpers shared across packages. `private: true` — never published, and **nothing under any package's `src/` may import it**. The only consumers are vitest suites.

No build step. `main` points at TypeScript source, because vitest transforms it anyway; a `dist/` would drag this package into the production build order for nothing. It is also outside `changeset:check`, which is correct — there is no published surface to version.

## WHY

Socket helpers had been copy-pasted into nine test files. The copies drifted apart, and the defect they shared had to be found three separate times before it was fixed once (#452).

## HOW

### Socket helpers are order-proof by construction

`ws.once('message', …)` only sees what arrives *after* it is attached, so waiting for a reply after sending the request loses it — and the symptom is a timeout pointing at the assertion, not at the ordering.

These record from the moment the socket opens and answer from the recording, so asking after the fact behaves exactly like asking before:

```ts
const ws = new WebSocket(url)
await waitForOpen(ws)          // recording starts here
ws.send(JSON.stringify({ type: 'device:boot', requestId, … }))   // required, or the relay drops it
await waitForType(ws, 'device:ready')   // fine whether it has landed yet or not
```

Matched messages leave the recording, so two waits for one type see two different messages.

**It is a queue, not a broadcast** — a change from the per-call listeners it replaced, where every
listener saw every message. A message now goes to exactly one waiter, so a pending `waitForMessage`
will take one that a concurrent `waitForType` was waiting for. Register the specific wait first, or
keep the two apart.

**`waitForOpen` is what starts the recording.** A socket that skips it still works — the other helpers begin recording on first use — but only from that call onwards, so anything earlier is genuinely gone. Route every socket through it.

### Assert absence with a barrier, not a timeout

`waitForTypeOrNull` exists, but a round-trip proves the same thing in milliseconds instead of guessing with a clock:

```ts
await barrier(ws)                                   // relay has finished everything sent before
expect(await waitForTypeOrNull(ws, 'device:ready', 0)).toBeNull()
```

Two sockets have no ordering between them — an agent's `device:ready` and a browser's `session:start` can be handled in either order. A test that depends on one preceding the other needs `barrier` on the **sending** socket.

### Narrowing is available but unused

`waitForType<RelayMessage>(…)` works. Almost nothing does it, because every package excludes
`src/__tests__` from `tsc` — a test file's types are stripped, never checked — so the narrowing
would be decorative. The parameter exists so that bringing tests under `tsc` is a config change
rather than a rewrite.

## HOW NOT

- Do not import this from production code. It is `private` and unbuilt; a `src/` import would fail at publish time, not at review time.
- Do not add a `dist/` or a build script. That is what keeps it out of the Docker build order.
- Do not reintroduce a local `waitForType` in a test file. That is the state this package replaced.

---
> Source: [jo-duchan/tapflow](https://github.com/jo-duchan/tapflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
