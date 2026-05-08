## testing

> Test writing rules for SGR Agent Core


# Test Writing Rules

## General Principles

1. **Coverage**: Aim for code coverage >90%
2. **One class at a time**: When creating new class, write tests for it immediately
3. **Isolation**: Each test must be independent
4. **Readability**: Test names should describe what is being tested
5. **Virtual environment**: Use `.venv` for running tests

## Test Structure

### Naming
- Test file: `test_<module_name>.py`
- Test class: `Test<ClassName>`
- Test method: `test_<what_is_tested>`

### Test Class Structure
```python
class TestAgentFactory:
    """Test suite for AgentFactory."""

    @pytest.mark.asyncio
    async def test_create_agent_from_definition(self):
        """Test creating agent from AgentDefinition."""
        # Arrange
        # Act
        # Assert
```

## Test Types

### Unit Tests
- Test individual modules in isolation
- Use mocks for dependencies (OpenAI client, external APIs)
- Fast and deterministic
- Use `@pytest.mark.asyncio` for async tests

### Integration Tests
- Test interaction between modules
- Test agent creation and execution flow
- Use real configuration but mocked LLM clients

### E2E Tests
- Test full agent execution cycle
- Use mocked LLM responses
- Verify agent state transitions

### Edge-case Tests
- Test edge cases (empty inputs, max iterations, etc.)
- Test error handling
- Test input validation

## Fixtures

### Using Fixtures
- Create fixtures in `tests/conftest.py` for reusable data
- Use `@pytest.fixture` for reusable data
- Use `scope="function"` for test isolation

### Common Fixtures
- `mock_openai_client` - Mocked AsyncOpenAI client
- `test_llm_config` - Test LLM configuration
- `test_prompts_config` - Test prompts configuration
- `test_execution_config` - Test execution configuration
- `create_test_agent()` - Helper function to create test agents

## Test Examples

### Unit Test for Agent Factory
```python
@pytest.mark.asyncio
async def test_create_agent_from_definition(self):
    """Test creating agent from AgentDefinition."""
    with (
        patch("sgr_agent_core.agent_factory.MCP2ToolConverter.build_tools_from_mcp", return_value=[]),
        mock_global_config(),
    ):
        agent_def = AgentDefinition(
            name="sgr_agent",
            base_class=SGRAgent,
            tools=[ReasoningTool],
            llm={"api_key": "test-key", "base_url": "https://api.openai.com/v1"},
            prompts={...},
            execution={},
        )
        agent = await AgentFactory.create(agent_def, task_messages=[{"role": "user", "content": "Test"}])

        assert isinstance(agent, SGRAgent)
        assert len(agent.task_messages) == 1
```

### Integration Test for Agent Execution
```python
@pytest.mark.asyncio
async def test_sgr_agent_full_execution_cycle(self):
    """Test full execution cycle of SGRAgent."""
    # Test agent creation and basic execution flow
```

## Assertions

### What to Check
- Return value types
- Agent state transitions
- Tool execution results
- Error messages
- HTTP status codes (for API tests)

### Using assert
- Use clear assert messages
- Check multiple aspects of result
- Use `isinstance()` for type checking

## Running Tests

### All Tests

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/ -v
```

**Windows (PowerShell):**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/ -v
```

**Windows (CMD):**
```cmd
.venv\Scripts\activate.bat
pytest tests/ -v
```

**Windows (Git Bash/WSL):**
```bash
source .venv/Scripts/activate
pytest tests/ -v
```

### With Coverage

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/ --cov=sgr_agent_core --cov-report=term-missing
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/ --cov=sgr_agent_core --cov-report=term-missing
```

### Specific Test

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/test_agent_factory.py::TestAgentFactory::test_create_agent_from_definition -v
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/test_agent_factory.py::TestAgentFactory::test_create_agent_from_definition -v
```

### Async Tests
- Use `pytest-asyncio` plugin (configured in `pytest.ini`)
- Mark async tests with `@pytest.mark.asyncio`
- Use `asyncio_mode = "auto"` in pytest.ini

## Mocking

### OpenAI Client Mocking
- Mock `AsyncOpenAI` client for all LLM calls
- Use `AsyncMock` for async methods
- Mock stream responses for streaming tests

### External Services
- Mock Tavily API calls
- Mock MCP server calls
- Mock file system operations when needed

## References

@pytest.ini
@tests/conftest.py
@code-style.mdc

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
