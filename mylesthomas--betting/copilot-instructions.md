## coding-princibles

> Core coding principles emphasizing functional clarity, fail-fast error handling, and readable, maintainable Python. Apply when writing or reviewing code.

# Coding Principles

Code should be functional, clear, concise, and easy to understand.

Before writing any code, the codebase should be fully read and understood to prevent writing unnecessary code, incompatible code, or duplicating existing code. Always read the existing code to know what functions you have access to, libraries you have already imported, etc.

## Standard Software Engineering Principles

- **DRY** (Don't Repeat Yourself)
- **Single Responsibility Principle**

## Function Guidelines

- Functions should generally not exceed **30 lines** (excluding comments and documentation).
  - If a function is growing too large, break it down into smaller, logically separated helper functions.
- Imports must exist **outside** of functions.
- Any function that is defined must also be subsequently called with the correct inputs to verify its validity.

## Error Handling

- `try` blocks are heavily discouraged.
- We prefer errors to be **raised** so that:
  - Key functionality underneath can be verified.
  - Bugs can be spotted early rather than hidden.
  - When importing a library, import it directly so we **fail fast** if there is an issue rather than hide it.

Rather than “defensive programming with conditionals,” we need **fail-fast** behavior (explicitly surfacing errors rather than hiding them), done with explainable assert statements.

## Code Philosophy

**Code should be:**
- Simple
- Elegant
- Easy to understand
- Complete

**Condition checking** is a requirement for all functions and should confirm that the inputs and outputs of each function are valid.

However, we don’t want ridiculous checks such as the following, because it hides errors. Either a value should be part of `locals` or it shouldn’t, so we should not have separate logic such as:

```python
if 'DEBUG_SIZE' in locals() and DEBUG_SIZE and len(work_order_notification_data) > DEBUG_SIZE:

---
> Source: [MylesThomas/betting](https://github.com/MylesThomas/betting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
