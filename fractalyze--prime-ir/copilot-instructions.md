## testing-guide

> Testing Rules


# Testing Rules

- **Framework**: Use gtest/gmock for C++ tests.
- **Coverage**: New features must include tests whenever applicable.
- **Completeness**: Always include boundary cases and error paths.
- **Determinism**: Tests must be deterministic and runnable independently (no hidden state dependencies).
- **Performance**: Add benchmarks for performance-critical code paths when appropriate.
- **Test-Driven Completeness**: When implementing features across dialects, use similar testing structures to ensure comprehensive coverage.

---
> Source: [fractalyze/prime-ir](https://github.com/fractalyze/prime-ir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
