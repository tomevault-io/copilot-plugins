## vision

> Product vision — what zsp is, who uses it, what success means


# zsp — Vision

## What zsp Is

`zsp` is the publishing tool for the Zapstore ecosystem. It takes an Android APK — from any source — and publishes it to Nostr relays as NIP-82 compliant software events.

It is used by app developers, CI pipelines, and the Zapstore indexer (`zindex`).

## Who Uses It

- App developers publishing their own apps to Zapstore
- CI/CD pipelines automating releases (GitHub Actions, etc.)
- `zindex` — the Zapstore batch indexer that runs `zsp` for hundreds of apps

## What Success Means

- A developer can publish their app in one command with minimal configuration
- CI pipelines can publish reliably without interactive prompts (`-y`, `--quiet`)
- Published events are always correct, verifiable, and relay-compatible
- The wizard guides first-time users to a working config

## Non-Goals

- `zsp` does not manage app stores or catalogs (that's the relay + webapp)
- `zsp` does not install apps (that's `zapstore` Flutter or `zapstore-cli`)
- `zsp` does not handle app discovery or search
- `zsp` does not support non-Android platforms

---
> Source: [zapstore/zsp](https://github.com/zapstore/zsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
