## flutter-boilerplate

> See **[AGENTS.md](AGENTS.md)** — the conventions and invariants for this

# CLAUDE.md

See **[AGENTS.md](AGENTS.md)** — the conventions and invariants for this
repository live there, in one file, so the two cannot drift apart.

Read it before changing code. The parts most often broken by accident:

- `core` must name no feature (`grep -ri dog_image lib/core` returns nothing)
- `dart analyze` must report **zero issues**, not merely zero errors
- Nothing generated is committed, and `env.g.dart` is never hand-edited
- `integration_test/` stays at the repo root, never inside `test/`
- Credentials never reach a log

Verify with `make analyze && make test` before reporting work as done.

---
> Source: [zeref278/flutter_boilerplate](https://github.com/zeref278/flutter_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
