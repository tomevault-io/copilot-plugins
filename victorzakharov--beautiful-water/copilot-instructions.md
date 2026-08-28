## beautiful-water

> - On Windows, if the sandboxed `apply_patch` helper fails with `helper_unknown_error` or its native

# Repository agent notes

- On Windows, if the sandboxed `apply_patch` helper fails with `helper_unknown_error` or its native
  wrapper corrupts multiline arguments, do not spend time retrying or debugging it. Edit only the
  scoped workspace files directly, then verify the result with `git diff`.

---
> Source: [VictorZakharov/beautiful-water](https://github.com/VictorZakharov/beautiful-water) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
