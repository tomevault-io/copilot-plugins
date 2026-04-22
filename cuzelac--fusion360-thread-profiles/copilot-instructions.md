## fusion360-thread-profiles

> Testing conventions for Ruby (Minitest/RSpec agnostic with Minitest examples)


### General principles

- Favor fast, deterministic tests; avoid external state and time dependencies.
- Test public APIs; avoid testing private methods directly.
- Make expectations specific; assert on values and types where relevant.

### Minitest style (fits this repo)

- Use `describe`/`it` or `class`/`def test_...` consistently.
- Use `before`/`setup` for common arrangement; avoid global state.
- Prefer `assert_raises(SpecificError)` with a block for error paths.

### Test organization

- Name files `*_test.rb`; mirror the source file names.
- Keep one subject per spec; separate behavior contexts with nested `describe`.
- Use fixtures/factories sparingly; build objects inline when simple.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cuzelac) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
