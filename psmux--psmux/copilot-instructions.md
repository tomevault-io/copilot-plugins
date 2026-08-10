## psmux

> - Never run psmux tests or commands that may create sessions in the default

# Agent Instructions

## Test Isolation

- Never run psmux tests or commands that may create sessions in the default
  namespace when working from an active psmux session.
- Some Rust tests create sessions internally and bypass the CLI `-L` option.
  Do not run `cargo test --all-targets` locally unless the process is inside a
  disposable Windows account, VM, or CI environment.
- Prefer safe targeted unit tests that do not start psmux servers, together
  with `cargo check`.
- Run runtime and integration checks with a unique namespace:
  `psmux -L <unique-test-namespace> ...`.
- Snapshot the default session list before and after runtime checks. Stop and
  investigate if it changes.
- Clean up only the test namespace with
  `psmux -L <unique-test-namespace> kill-server`.
- Never use bare `psmux kill-server` for test cleanup because it affects every
  namespace.
- Run the full test suite only in CI or another disposable Windows environment
  where user sessions cannot be affected.

---
> Source: [psmux/psmux](https://github.com/psmux/psmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
