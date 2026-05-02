## testing

> > Apply when working with test files.

# Windsurf Rules - Testing

> Apply when working with test files.

## Structure

- Use AAA pattern: Arrange → Act → Assert
- One assertion concept per test
- Descriptive names: "should [action] when [condition]"

## Philosophy

- Test behavior, not implementation
- Mock external dependencies (APIs, DB)
- Prefer integration tests for critical paths

## Coverage

- >80% on business logic
- 100% on security-critical code
- Skip trivial getters/setters

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
