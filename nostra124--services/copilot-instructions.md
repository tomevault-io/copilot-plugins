## services

> > Mirrors `CLAUDE.md.foundation`, specialised for

# `services` — developer notes

> Mirrors `CLAUDE.md.foundation`, specialised for
> educational `services`.

## 1. Scope

`services` is the multi-backend supervision frontend.
Its scope is the shared verb surface
(create / start / stop / restart / status / monitor /
hook / list) across daemontools / runit / systemd-user /
launchd.

Out of scope: the supervisors themselves; container
runtime supervision (that's the cpk-base layer's
runit-as-PID-1 setup).

## 2. Repo conventions

Standard rpk per-package: `bin/services` dispatcher
with `command:<verb>` functions. Backend-specific
adaptation lives in inline case-on-platform blocks per
verb; this is intentional — sharing a backend-adapter
library would obscure which platform exhibits which
behaviour.

Educational package: ships vendored references under
`share/doc/services/standards/` and a walkthrough at
`docs/services-walkthrough.md`.

## 3. Issue authoring

Same as `CLAUDE.md.foundation`. **Bugs come before
features at the same priority level.**

## 4. The no-shared-lib policy

`services` calls only `account` at runtime (for
platform detection). Each verb's backend-adaptation
code is inline — not pulled into a backend-helpers
library — so a reader sees the full mapping per verb.

The temptation to factor "all systemd-user behaviour"
into a sibling file is **wrong** for educational
clarity; keep it inline.

## 5. What is intentionally duplicated

- **Platform-detection blocks.** Same `case $(account
  platform) in linux*) … macos*) …` pattern in every
  verb. Intentional; readers see the full per-verb
  mapping.
- **Per-supervisor command construction** (e.g. `sv
  start <n>` vs `systemctl --user start <n>.service`).
  Inline per verb.

## 6. Consumers

End users running long-running daemons; cpk-base for
container PID-1; cluster plugins that need per-node
service supervision.

## 7. Build / install

`./configure && make install`. Stow-based.

## 8. Versioning

Semver. `tests/unit/services.bats` is the contract.

---
> Source: [nostra124/services](https://github.com/nostra124/services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
