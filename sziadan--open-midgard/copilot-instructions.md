## reference-lookup-rules

> Prefer Ref sources for implementation details


# Reference Lookup Rules

- When additional implementation detail is needed, check the `Ref` folder before guessing.
- Use `Ref` as the decompiled original client reference when recreating client behavior.
- Use `Ref/eAthena_src_2011` and `Ref/RunningServer` when verifying server-side packet logic, protocol behavior, or emulator expectations.
- Use `Ref/GRF-Content` when verifying unpacked GRF asset names, paths, or data layout.
- Prefer conclusions grounded in these reference sources over unsupported assumptions.
- If reference sources disagree, call out the mismatch explicitly and explain which source is being followed.

---
> Source: [Sziadan/open-midgard](https://github.com/Sziadan/open-midgard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
