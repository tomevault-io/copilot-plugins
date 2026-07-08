## tennis-origination

> - **BE CONCISE**: One sentence per issue


---

## 🔍 Code Review Guidelines

### Core Rules

- **BE CONCISE**: One sentence per issue
- **BE SPECIFIC**: Reference exact lines/functions
- **BE OBJECTIVE**: Focus on correctness and maintainability
- **STAY IN SCOPE**: Only review what the PR changes
- **NO FORCED FEEDBACK**: Only comment on real issues
- **APPROVE WHEN APPROPRIATE**: Say *"This looks good!"* if no issues

---

## 🧱 Core Engineering Principles

### 1. Clarity Over Cleverness

- Prefer simple, readable code over clever solutions
- Optimize for maintainability, not novelty

### 2. Fail Fast, Fail Clearly

- Validate inputs early
- Provide actionable error messages
- Catch issues as early as possible (types, validation, assertions)

### 3. Separation of Concerns

- Each module/function should have a single responsibility
- Avoid tightly coupled logic

### 4. Configuration Over Hardcoding

- Avoid embedding environment-specific values in code
- Use environment variables or config systems

---

## 🧬 Type Safety & Validation

### Type Annotations

- All function signatures should include type hints
- Use modern Python syntax (`str | None`)
- Use typed collections (`list[str]`, `dict[str, int]`)

### Data Validation

- Use structured validation (e.g., Pydantic or equivalent) when applicable
- Define constraints explicitly (ranges, formats, etc.)
- Prefer immutable data structures where possible

---

## ⚠️ Error Handling

- Use clear, domain-relevant exceptions where appropriate
- Avoid generic `Exception` unless truly necessary
- Ensure error messages are actionable

---

## ⚙️ Configuration Management

- Centralize configuration
- Validate configuration values
- Avoid scattering environment logic across the codebase

---

## 🧹 Code Quality Standards

### Formatting & Linting

- Use automated tools where available (e.g., Ruff, Black, MyPy)
- Maintain consistent import ordering:
  1. Standard library
  2. Third-party
  3. First-party

### Naming Conventions

- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`

---

## 🧪 Testing Philosophy

### General Expectations

- New logic should be covered by tests
- Focus on **behavior**, not implementation details

### Test Quality

- Use descriptive test names
- Cover:
  - Happy path
  - Failure cases
  - Key edge cases

### Best Practices

- Use AAA pattern (Arrange, Act, Assert)
- Mock external dependencies where appropriate
- Prefer reusable test fixtures/factories

---

## 📚 Documentation

### Code

- Prefer self-documenting code over excessive comments
- Use docstrings for public or complex functions

### Project

- Update documentation when behavior or architecture changes
- Document non-obvious design decisions

---

## 📊 Observability (When Applicable)

- Use structured logging
- Avoid logging sensitive data
- Include identifiers for tracing when useful

---

## 🚀 Performance (When Relevant)

- Optimize only when necessary
- Cache expensive operations when appropriate
- Avoid premature optimization

---

## 🔐 Security Basics

- Validate all inputs
- Do not expose sensitive data
- Use environment variables for secrets

---

## 🔄 Async & Concurrency (When Applicable)

- Use async patterns for I/O-bound work
- Avoid mixing sync/async unnecessarily
- Handle errors properly in concurrent code

---

## 🚫 Anti-Patterns to Avoid

### Code

- God classes (too many responsibilities)
- Deep nesting (>3–4 levels)
- Magic numbers
- Duplicate logic
- Long parameter lists

### Architecture

- Circular dependencies
- Leaky abstractions
- Premature optimization

---

## ✅ Final Note

These are guidelines, not rigid rules.

Prioritize:
1. Correctness
2. Clarity
3. Maintainability

Use judgment based on the context of the change.

---
> Source: [mattholden/tennis-origination](https://github.com/mattholden/tennis-origination) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
