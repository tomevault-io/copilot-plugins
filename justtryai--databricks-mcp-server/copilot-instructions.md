## testing

> Testing Guidelines

# Testing Standards

## Framework and Requirements
- Use pytest for all tests
- Minimum code coverage: 80%
- Excluded from coverage: scripts/*, examples/*

## Test Types
- Unit tests: Test individual functions and classes in isolation
- Integration tests: Test components working together
- MCP protocol tests: Verify correct implementation of the MCP protocol

## File Organization
- Test files should be named `test_*.py`
- Each file in `src/` should have a corresponding test file in `tests/`
- Structure tests to mirror the source directory structure

## Best Practices
- Use fixtures to set up test environments
- Use parametrized tests for multiple test cases
- Mock external dependencies (especially API calls)
- Keep tests fast and independent of each other

## Example
```python
import pytest
from unittest.mock import AsyncMock, patch

from src.api.cluster import list_clusters

@pytest.fixture
def mock_client():
    client = AsyncMock()
    client.make_request.return_value = {"clusters": [{"cluster_id": "abc123"}]}
    return client

@pytest.mark.asyncio
async def test_list_clusters(mock_client):
    """Test that list_clusters returns expected format."""
    result = await list_clusters(mock_client)
    
    assert isinstance(result, list)
    assert len(result) == 1
    assert result[0]["cluster_id"] == "abc123"
    mock_client.make_request.assert_called_once()
```

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
