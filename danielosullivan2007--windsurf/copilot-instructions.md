## windsurf

> 1. Follow PEP 8 guidelines for code formatting and structure.


<python_coding>
  <style>
    1. Follow PEP 8 guidelines for code formatting and structure.
    2. Use 4 spaces per indentation level; avoid tabs.
    3. Limit lines to a maximum of 79 characters.
    4. Group imports as: standard library, third-party, local application/library-specific.
    5. Insert a blank line between each group of imports.
  </style>

  <naming>
    1. Use `snake_case` for functions and variables.
    2. Use `PascalCase` for class names.
    3. Use `UPPER_CASE` for constants.
    4. Avoid single-character variable names, except for counters or iterators.
  </naming>

  <documentation>
    1. Include docstrings for all public modules, functions, classes, and methods.
    2. Use triple double quotes (`"""`) for docstrings.
    3. Keep docstrings concise yet descriptive, explaining purpose and usage.
  </documentation>

  <structure>
    1. Define one class per module; group related functions together.
    2. Avoid deep nesting of code blocks; aim for flat and readable structures.
    3. Separate top-level function and class definitions with two blank lines.
    4. Separate method definitions inside a class with one blank line.
  </structure>

  <error_handling>
    1. Use specific exceptions in `try-except` blocks; avoid bare `except` clauses.
    2. Limit the scope of `try` blocks to the smallest possible code that might raise an exception.
    3. Avoid using exceptions for control flow.
  </error_handling>

  <type_annotations>
    1. Use type hints for function arguments and return types.
    2. Utilize the `typing` module for complex type annotations.
    3. Maintain consistency in type annotations throughout the codebase.
  </type_annotations>

  <comments>
    1. Explain the "why" behind complex or non-obvious code.
    2. Keep comments up-to-date with code changes.
    3. Avoid redundant comments that state the obvious.
  </comments>

  <tools>
    1. Use linters like `flake8` or `pylint` to enforce coding standards.
    2. Employ code formatters like `black` to maintain consistent code style.
    3. Integrate these tools into the development workflow for continuous code quality checks.
  </tools>
</python_coding>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielosullivan2007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
