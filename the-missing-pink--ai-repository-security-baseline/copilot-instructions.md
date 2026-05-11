## 03-testing

> Testing requirements — test-before-commit policy, coverage expectations, and test quality standards.


# Testing Requirements

## Test-Before-Commit Policy

- Run the full test suite before every commit.
- Run linting and type checking before every commit.
- Never skip tests with --no-verify, skip, xfail, or equivalent.
- Never delete or disable existing tests to make new code pass.

## Coverage

- All new functions, methods, and endpoints must have tests.
- Bug fixes must include regression tests.
- Test edge cases, error paths, and boundary conditions.

## Quality

- Tests must be deterministic: no random data, no time-dependent assertions.
- Use mocks and stubs for external services.
- Test names must describe the scenario and expected behavior.
- Follow Arrange-Act-Assert pattern.

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
