## swolectl

> This repository is an independent implementation of a controller for compatible

# Project instructions

This repository is an independent implementation of a controller for compatible
strength-training hardware.

## Protocol documentation

- Keep the repository limited to its own source, documentation, tests, and
  user-created media.
- Do not include or commit copyrighted vendor property, source code, binaries,
  firmware, applications, packet captures, extracted assets, or other vendor
  artifacts.
- Record evidence and confidence for every field in `docs/protocol.md`.
- Never guess a motor-control field. Unknown fields remain named `unknown_*` or
  unsupported until their behavior is verified.
- Do not commit device identifiers, secrets, or user data.

## Development

- Use `uv` for environments, dependencies, and commands.
- Support Python 3.11 and newer.
- Run `uv run pytest`, `uv run ruff check .`, and `uv run mypy` before handoff.
- Keep the protocol layer independent from pyserial so it can be tested without
  hardware.
- Hardware tests must be opt-in and must default to receive-only behavior.

## Safety

- Motor commands require explicit enablement through `SafetyPolicy`.
- Resistance must be bounded and finite.
- Closing the high-level controller should attempt to disable resistance.
- Never add automatic replay of arbitrary raw commands.
- Clearly distinguish observed facts, inference, and unknown behavior.

---
> Source: [d4l3k/swolectl](https://github.com/d4l3k/swolectl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
