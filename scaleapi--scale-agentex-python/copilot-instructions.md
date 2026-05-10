## 50-tests-and-mocking

> Testing workflow and mock server details


Testing:

- Run tests with `rye run pytest` or `./scripts/test`
- To run a specific test: `rye run pytest path/to/test_file.py::TestClass::test_method -v`
- A mock server is automatically started for tests on port 4010

When writing tests:

- Prefer deterministic unit tests that do not depend on external services
- Use the mock server and fixtures provided in the repository

---
> Source: [scaleapi/scale-agentex-python](https://github.com/scaleapi/scale-agentex-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
