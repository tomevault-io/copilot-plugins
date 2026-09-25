## tokenledger

> TokenLedger is a renderer-independent DeepSeek Harness domain plugin. Keep the

# AGENTS.md

TokenLedger is a renderer-independent DeepSeek Harness domain plugin. Keep the
accounting direction as durable Harness session logs -> TokenLedger store ->
`usagePayload()` -> bounded renderer views. The Web client and the in-package
Blue adapter consume that domain truth rather than folding session events.

Blue is an optional renderer inside the single `dsh-tokenledger` npm package.
The root Cordis `apply()` owns one internal dashboard controller and passes it
directly to the Blue adapter in the same Fiber. Do not publish that controller
on `ctx`, add a second Cordis plugin row, create another npm package, or restore
the public `ctx.tokenLedgerV1` boundary. The internal controller keeps practical
collection bounds, revision consistency, abort/stale fencing, credential
filtering, and unload-safe late-result behavior without a hostile third-party
clone/descriptor protocol.

`ctx.tokenLedger` is a separate legacy/deprecation boundary. Preserve its object
identity and its exact `store`, `sweep`, `totals`, `byDay`, `byModel`, `bySite`,
`sites`, `diagnostics`, and `reindex` behavior. Do not silently narrow or replace
it. New external consumers do not receive an implicit replacement API; design
an explicit public contract only when a real external consumer requires one.

Mutable caches belong to one `apply()` instance. The Cordis plugin must use
`createNewApiWalletReader()`; the module-level wallet helper exports exist only
for compatibility. Cleanup order is Blue/controller request abort, owned cache
dispose, then store close. Never expose `LedgerStore` through the dashboard
controller.
Usage-derived notifications retain fingerprint deduplication. Settings
mount/unmount, initial/watch values, registration failure, and legacy settings
writes must notify the internal controller so configuration and writability
changes advance its revision even when usage rows are unchanged. Notify again
after `settingsScope` is assigned; an earlier initial-value callback does not
prove that writes are ready.

Keep the existing Web UI and loopback HTTP API as the golden/plain fallback.
Blue-specific interaction and renderer code belongs under `src/blue/`; it must
remain isolated from accounting and store ownership. With no Blue host, retain
the legacy `/tokenledger` command. With Blue mounted, expose exactly one
renderer-native `/tokenledger` command and restore the legacy command if Blue
unloads or fails to register.

Run `npm test` and `npm pack --dry-run --json` from the repository root. The
package intentionally has no build step. New exports must stay under `src/`, be
listed in `package.json`, and be covered by `test/packaging.test.js`.

---
> Source: [zh667/TokenLedger](https://github.com/zh667/TokenLedger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
