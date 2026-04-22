## ai-sdlc-agent-poc

> Testing Standards for FastAPI projects

# Testing Standards for FASTAPI Projects

## Core Principles
- Test behaviour, not implementation.
- Prefer integration tests over isolated unit tests. Testing multiple units together in a functional way
- Mock external dependencies only, at the lowest level (e.g. database operations, API calls, etc).
- Prioritise clarity and readability.
- Follow Given-When-Then pattern with inline comments

## Test Cases
- **Happy Path**: Ensure that valid data produces expected responses and database state.
- **Error Cases**: Cover invalid input (400), resource not found (404), server errors (500) and dependency failures e.g. database failures

## Best Practices
- **Organisation**: Group tests by endpoint or operation with clear comment headers (e.g. "# Test Cases - Create").
- **Naming**: Use descriptive test names (e.g. `test_operation_scenario_expected`).
- **Fixtures**: Use fixtures for common test data and mocks.
- **Assertions**: Validate status codes first, then response bodies.
- **Async Testing**: Use AsyncClient for async endpoints and AsyncMock for async operations.
- **Look for existing test patterns first**:
  - Check similar test files for patterns
  - Review conftest.py for established mocking approaches
  - Reuse existing test data from test_data.py

## Test File Structure
Each test file should follow this structure:
1. File docstring explaining purpose
2. Imports (stdlib, third-party, local)
3. Setup fixtures (autouse and specific)
4. Test cases grouped by operation
5. Helper functions (if needed)

## Database Mocking Approach
If the test require MongoDB - reference [03_mongodb_mocking.mdc](mdc:.cursor/rules/03_mongodb_mocking.mdc)

## Background Process Mocking
When testing endpoints that spawn background processes:

1. Always mock at the service level where the Process is instantiated
```python
# CORRECT - Mock where Process is created
with patch('app.services.standard_set_service.Process'):
    response = await async_client.post(...)

# INCORRECT - Mock at too high a level
with patch('multiprocessing.Process'):
    response = await async_client.post(...)
```

2. Common pitfalls to avoid:
   - Don't mock at the global multiprocessing level
   - Don't let test processes persist after test completion
   - Remember to mock in all test cases that trigger the service

### Test Data Management
1. ALL test data should be in test_data.py
2. Use `create_db_document` for generating test documents
3. Include all required fields, timestamps, and ObjectId
4. Provide helper functions for each document type (e.g. `create_classification_test_data`)
5. Use fixtures for input data validation (e.g. `valid_classification_data`)

## Test Structure Pattern
Each test should follow this structure:
```python
async def test_operation_scenario_expected(
    async_client,
    mock_database_setup,
    mock_collections,
    test_data_fixtures
):
    # Given: Setup context with clear comments
    collection = mock_collections
    repo = Repository(collection)
    service = Service(mock_database_setup, repo)
    app.dependency_overrides[get_service] = lambda: service

    # When: Action is performed
    response = await async_client...

    # Then: Verify results
    assert response.status_code == status.HTTP_XXX_XXX
    data = response.json()
    assert ... # Additional assertions
```

## Tools & Frameworks
- **pytest** with **pytest-asyncio** for async tests.
- **pytest-cov** for coverage (minimum 90%).
- **httpx** AsyncClient for async HTTP testing.
- FastAPI's **TestClient** for synchronous tests.

## FastAPI Testing Guidelines
- Configure TestClient/AsyncClient with the correct `base_url` and manage lifespan events.
- Override dependency providers (not the implementations) and reset overrides between tests.
- Validate responses against both status codes and Pydantic models.
- Use proper async/await patterns throughout tests.

## Test Directory Structure
```bash
tests/
├── conftest.py   # Fixtures and configuration
├── integration/  # Integration tests
├── unit/        # Unit tests
└── utils/       # Utility functions and test data
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DEFRA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
