## ut-plugin-payment-demo

> One plugin per repo (docs repo ADR-0009). This is the **demo/simulated

# ut-plugin-payment-demo — rules for working in this repo

One plugin per repo (docs repo ADR-0009). This is the **demo/simulated
card terminal** (`canonical_type: payment`, `runtime: wasm`) and the
reference for real terminal plugins.

- Deterministic outcomes are the contract: pence `.13` declined, `.99`
  timeout, else approved. Never make them random — E2E tests and training
  scripts depend on them.
- The wasm module is plain Go `GOOS=wasip1 GOARCH=wasm` (scripts/build.sh);
  host functions imported from module `ut` (see docs repo
  `reference/plugin-host-functions.md`); `storage` permission is declared
  in the manifest — keep code and manifest in sync.
- Release = tag `v<version>` matching manifest.json; CI runs
  build → validate → package → publish → (dev) auto-approve. No release
  logic in YAML beyond orchestrating `scripts/*.sh`.
- Standards & decisions live in the docs repo (`adr/`, ADR-0007
  document-first). Behaviour changes update the README in the same session.

---
> Source: [universaltill/ut-plugin-payment-demo](https://github.com/universaltill/ut-plugin-payment-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
