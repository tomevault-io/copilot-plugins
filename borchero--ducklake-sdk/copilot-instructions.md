## ducklake-sdk

> - Never use classes for pytest, but only free functions

# Context

## Testing

- Never use classes for pytest, but only free functions
- Do not put `__init__.py` files into test directories
- Tests should not have docstrings unless they are very complicated or very specific, i.e. warrant a description beyond
  the test's name
- All tests should follow the arrange-act-assert pattern. The respective logical blocks should be distinguished via
  code comments as follows:

  ```python
  def test_method() -> None:
      # Arrange
      ...

      # Act
      ...

      # Assert
      ...
  ```

- If two or more tests are structurally equivalent, they should be merged into a single test and parametrized with
  `@pytest.mark.parametrize`
- If at least two tests share the same logic in the "arrange" step, the respective logic should be extracted into a
  fixture

---
> Source: [borchero/ducklake-sdk](https://github.com/borchero/ducklake-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
