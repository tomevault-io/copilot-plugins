## intentra

> Testing requirements and strategy


# Testing Requirements

- Unit test pure logic:
  - Diff parsing + hunk hashing.
  - Schema validation + business validation.
  - Patch generation correctness.
- For git executor:
  - Integration tests using a temp git repo.
  - Ensure abort restores index on failure.
- Mock LLM client in tests; never hit network.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crvgilbertson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
