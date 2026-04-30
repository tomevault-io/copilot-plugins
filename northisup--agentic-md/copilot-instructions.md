## agentic-md

> Testing


- TDD: write failing test first, then implement
- Unit tests are inline (`#[cfg(test)] mod tests`) in each module
- Integration tests in `tests/integration/` exercise the full `run()` function
- Use `tempfile::tempdir()` for filesystem tests — never write to the real project dir in tests
- Test both the happy path and edge cases (empty input, missing files, conflicts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NorthIsUp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
