## ai-sdlc-agent-poc

> Mongo DB Database Mocking

# Best Practices for Mocking MongoDB in Tests

This document outlines the recommended rules and practices for mocking MongoDB interactions in our codebase. It builds on our current approach and includes additional suggestions for improved test isolation, clarity, and reliability.

Firstly, ensure you scan the entire testing patterns to determine if Mongo is already being mocked, rather than repeat the implmentation!

## 1. **General Principles**

- **Isolate Your Tests:**
  Always use mocks to replace real MongoDB connections, ensuring tests run in isolation and without side effects.

- **Use Dependency Injection:**
  Inject mocked repositories and services into your application (e.g., via FastAPI dependency overrides) to avoid accidental use of production code.

- **Consistent Naming Conventions:**
  Name fixtures and mocks clearly (e.g., `mock_database_setup`, `setup_list_cursor`) to make it easy to trace their usage.

## 2. **Global Database Patching**

- **Patch Key Entry Points:**
  - Patch `init_database` to return a mocked database.
  - Replace the actual MongoDB client (`motor.motor_asyncio.AsyncIOMotorClient`) with a `MagicMock`.
  - Patch helper functions like `get_database` to ensure all database interactions use the mock.

- **Ensure Complete Isolation:**
  Confirm that any code initializing a connection to MongoDB uses the patched versions so no live database is ever contacted during tests.

## 3. **Collection and Cursor Mocks**

- **Asynchronous Collections:**
  Use `AsyncMock` for collection methods that perform async operations (e.g., `find`, `insert_one`).

- **Cursor Chaining:**
  When mocking cursor chains:
  - Make sure that methods such as `sort()` return the same cursor mock.
  - Example:
    ```python
    mock_cursor.sort = MagicMock(return_value=mock_cursor)
    ```

- **Property Patching:**
  Use `PropertyMock` to simulate attribute access (e.g., linking a collection's `.database` attribute to the mocked database):
  ```python
  type(collection).database = PropertyMock(return_value=mock_db)
  ```

- **Dynamic Collection Resolution:**
  If you have multiple collections, consider using a dictionary with a side_effect to route collection names correctly:
  ```python
  mock_db.get_collection = MagicMock(side_effect=lambda name: {
      "standards": standards_collection,
      "classifications": classifications_collection,
      # add other collections as needed
  }[name])
  ```

## 4. **Operation Mocks**

- **Find and Cursor Operations:**
  - Mock the find method to return a cursor whose to_list method returns your predefined list of documents.
  - Use a helper fixture like setup_list_cursor to standardize this behavior.

- **CRUD Operations:**
  - Insert: Mock insert_one to return an object with an inserted_id (consider using a helper fixture like setup_mock_result).
  - Update & Delete: Ensure update_one and delete_one return objects with modified_count, matched_count, or deleted_count.

- **Error Simulation:**
  - Leverage side_effect to simulate exceptions (e.g., side_effect=Exception('Database error')).
  - Always test both success and error scenarios.

- **Explicit Assertions:**
  - Use assertions such as assert_called_once_with to verify that mocked methods are called with the expected arguments.

## 5. **Service and Dependency Injection**

- **Mocking Services:**
  - Wrap collection mocks in repository classes and inject them into services.

- **Dependency Overrides:**
  - In FastAPI or similar frameworks, override dependencies to ensure the application uses mocks during tests:
  ```python
  app.dependency_overrides[get_service] = lambda: service
  ```

## 6. **Standardized Test Fixtures**

- **Autouse Fixtures for Setup/Teardown:**
  - Use autouse fixtures (e.g., setup_and_teardown) to automatically manage test state and clear dependency overrides.
  - Consider resetting mocks (via reset_mock()) between tests if the mock's state might leak across tests.

- **Custom Result Fixtures:**
  - Create helper fixtures to standardize the return values for CRUD operations.

## 7. **Environment and Test Data Management**

- **Mock Environment Variables:**
  - Use monkeypatch fixtures to ensure your tests run with a controlled environment:
  ```python
  @pytest.fixture(autouse=True)
  def mock_env_vars(monkeypatch):
      monkeypatch.setenv("MONGODB_URL", "mongodb://test:27017")
      monkeypatch.setenv("DATABASE_NAME", "test_db")
  ```

- **Consistent Test Data:**
  - Utilize shared utilities (e.g., in tests/utils/test_data.py) to create standardized test data for different collections.

- **Pre-populated Collections:**
  - For integration tests, use fixtures like mock_collections_with_data to simulate collections with initial data.

## 8. **Extending and Maintaining Mocks**

- **Adding New Collections/Operations:**
  - Follow existing patterns:
    - Use AsyncMock for asynchronous operations.
    - Apply PropertyMock for property access.
    - Ensure consistency in the return types and structure of your mocks.

- **Document Side Effects:**
  - Clearly document when and why you use side_effect to simulate errors.

- **Use Additional Tools if Needed:**
  - Consider using plugins like pytest-mock for cleaner syntax and more readable assertions.

## 9. **Additional Good Practices**

- **Test Coverage:**
  - Ensure all CRUD methods and error scenarios are covered by your tests.

- **Realism in Mocks:**
  - Mimic actual MongoDB behavior as closely as possible, including asynchronous behavior, to avoid surprises in production.

- **Refactoring and Maintenance:**
  - Regularly review and refactor your mocking strategy to align with any changes in the production code or database client library.

- **Logging & Debugging:**
  - Consider adding logging or verbose messages in fixtures to help diagnose test failures related to mocking.


## 10. **Additional Test Fixtures and Patterns**

### Collection Relationships and State
When collections have relationships (e.g. foreign keys):
```python
@pytest.fixture
async def mock_collections():
    """Setup collections with relationships."""
    primary_collection = AsyncMock()
    related_collection = AsyncMock()

    # Setup database relationship
    mock_db = AsyncMock()
    mock_db.get_collection = MagicMock(return_value=related_collection)
    type(primary_collection).database = PropertyMock(return_value=mock_db)

    # Assign to database setup
    mock_database_setup.primary = primary_collection
    mock_database_setup.related = related_collection

    return primary_collection, related_collection
```

### Test State Management
Always include an autouse fixture to manage test state:
```python
@pytest.fixture(autouse=True)
async def setup_and_teardown():
    """Setup and teardown for each test."""
    # Setup
    yield
    # Teardown - clear dependency overrides
    app.dependency_overrides = {}
```

### Collection Mocking Pattern
Use a fixture to set up collection mocks:
```python
@pytest.fixture
async def mock_collections():
    """Setup mock collections for tests."""
    collection = AsyncMock()

    # Mock the database property for nested collections
    mock_db = AsyncMock()
    type(collection).database = PropertyMock(return_value=mock_db)

    return collection
```

### Service Setup Pattern
Each test should set up its service with mocked dependencies:
```python
# Setup repository and service
repo = Repository(collection)
service = Service(mock_database_setup, repo)
app.dependency_overrides[get_service] = lambda: service
```

### Collection Operation Mocking
Mock MongoDB operations directly on the collection object:
```python
# Mock find_one operation
collection.find_one = AsyncMock(return_value=mock_doc)

# Mock find operation with cursor
mock_cursor = AsyncMock()
mock_cursor.to_list = AsyncMock(return_value=[mock_doc])
collection.find = MagicMock(return_value=mock_cursor)

# Mock insert operation
collection.insert_one = AsyncMock()

# Mock delete operation with count
mock_result = AsyncMock()
mock_result.deleted_count = 1
collection.delete_one = AsyncMock(return_value=mock_result)

# Mock update operation
mock_result = AsyncMock()
mock_result.modified_count = 1
collection.update_one = AsyncMock(return_value=mock_result)

# Mock error cases
collection.operation = AsyncMock(side_effect=Exception("Database error"))
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DEFRA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DEFRA)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
