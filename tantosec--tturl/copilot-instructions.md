## tturl

> Follow [`CONTRIBUTING.md`](CONTRIBUTING.md) for the supported API, language,

# Repository guidance

Follow [`CONTRIBUTING.md`](CONTRIBUTING.md) for the supported API, language,
validation, and contribution workflow.

## Architecture

- `stats` contains domain-free statistical primitives.
- `internal/ranking` owns inference from noisy ordinal comparisons.
- `internal/timing` owns request-duration transport and scheduling.
- `tth2` owns reusable HTTP/2 mechanics.
- `cmd/tturl` owns the command-line interface.

---
> Source: [tantosec/tturl](https://github.com/tantosec/tturl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
