## testing-standards

> - **TDD required** - All new features MUST include pytest tests. No exceptions. Run: `UV_CACHE_DIR=.uv-cache uv run pytest`

# Testing Standards

- **TDD required** - All new features MUST include pytest tests. No exceptions. Run: `UV_CACHE_DIR=.uv-cache uv run pytest`
- **Test naming** - `def test_{what}_{scenario}():` with one-line docstring
- **Arrange-Act-Assert structure** - Organize tests clearly with comments for each section
- **Mock external dependencies** - Never make real API calls unless purely for testing API functionality, do not make database queries, or file I/O. Use `AsyncMock` and test implementations
- **Use @pytest.mark.asyncio** - For async tests
- **Required coverage** - Happy path, error cases, edge cases, integration tests
- **All tests must pass before committing** - No broken tests in main branch
- **No fake test passes** - Never use `pytest.skip()`, `pass`, or `assert True` to fake passing tests. Write real assertions that validate actual behavior

---
> Source: [miniverse-ai/miniverse](https://github.com/miniverse-ai/miniverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
