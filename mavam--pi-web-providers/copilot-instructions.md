## pi-web-providers

> This repository contains **pi-web-providers**, a pi extension for configurable web access.

# pi-web-providers

This repository contains **pi-web-providers**, a pi extension for configurable web access.

It lets pi expose web-related tools through interchangeable providers instead of tying each capability to a single backend.

For the full product overview, supported capabilities, configuration, and provider-specific details, see `README.md`.

## Repository map

- `src/` — extension implementation and provider integrations
- `test/` — automated coverage for configuration, provider behavior, and tool execution
- `examples/` — example wrapper-based setups and reference integrations
- `example-config.json` — minimal example configuration
- `scripts/` — project scripts such as live smoke testing
- `changelog/` — release notes and changelog entries

## Guidelines

- Keep `package.json` metadata, especially `description` and `keywords`, in sync with the README.
- When user-facing behavior changes, update `README.md` alongside the code.
- When changing configuration, routing, or provider behavior, update or add tests in `test/`.
- Keep examples and sample configuration aligned with the current behavior.
- The changelog is managed via the `tenzir-ship` tool; do not edit `changelog/` by hand.

---
> Source: [mavam/pi-web-providers](https://github.com/mavam/pi-web-providers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
