## implementation-order

> Implementation order for new features - one class at a time


# Implementation Order: One Class at a Time

## "One Class at a Time" Principle

When adding new functionality, follow **"one class at a time"** principle, starting from lower architecture layers.

## Architecture Layers (Bottom to Top)

1. **Layer 1**: Base classes (BaseAgent, BaseTool, Models)
2. **Layer 2**: Configuration and Registry
3. **Layer 3**: Factory and Services
4. **Layer 4**: Agent Implementations
5. **Layer 5**: Tools
6. **Layer 6**: Server and API

## Implementation Steps

### 1. Determine Layer
Determine which layer the new functionality belongs to (see @architecture.mdc)

### 2. Write Test First
- Create test file `tests/test_<module_name>.py`
- Write minimal unit tests for class
- Test must **fail** (red) because class doesn't exist
- Run test:
  - **Linux/macOS**: `source .venv/bin/activate && pytest tests/test_<module_name>.py::test_name -v`
  - **Windows**: `.venv\Scripts\activate && pytest tests/test_<module_name>.py::test_name -v`
- Verify test fails for correct reason

### 3. Implement Class
- Create class with minimal implementation
- Use type hints
- Add docstrings in English
- Follow rules from @code-style.mdc
- Place in appropriate layer directory

### 4. Verify Test Passes
- Run test:
  - **Linux/macOS**: `source .venv/bin/activate && pytest tests/test_<module_name>.py::test_name -v`
  - **Windows**: `.venv\Scripts\activate && pytest tests/test_<module_name>.py::test_name -v`
- Test must **pass** (green)
- Make sure implementation is correct

### 5. Run All Tests
- Execute:
  - **Linux/macOS**: `source .venv/bin/activate && pytest tests/ -v`
  - **Windows**: `.venv\Scripts\activate && pytest tests/ -v`
- All tests must pass
- Fix any regressions

### 6. Run Linter
- Execute:
  - **Linux/macOS**: `source .venv/bin/activate && pre-commit run -a`
  - **Windows**: `.venv\Scripts\activate && pre-commit run -a`
- Fix all linting errors
- Repeat until all checks pass

### 7. Move to Next Layer
Only after lower layer class is ready and tested, move to upper layer classes.

## Example: Adding New Tool

### Step 1: Write Test
```python
# tests/test_custom_tool.py
@pytest.mark.asyncio
async def test_custom_tool_execution():
    """Test CustomTool execution."""
    tool = CustomTool(param="value")
    result = await tool(context, config)
    assert result == "expected_result"
```

### Step 2: Run Test (Should Fail)

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/test_custom_tool.py::test_custom_tool_execution -v
# Expected: FAILED - CustomTool doesn't exist
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/test_custom_tool.py::test_custom_tool_execution -v
# Expected: FAILED - CustomTool doesn't exist
```

### Step 3: Implement Tool (Layer 5)
```python
# sgr_agent_core/tools/custom_tool.py
class CustomTool(BaseTool):
    """Custom tool for specific functionality."""
    tool_name: str = "custom_tool"
    description: str = "Does custom thing"

    param: str

    async def __call__(self, context: AgentContext, config: AgentConfig) -> str:
        """Execute custom tool."""
        return "expected_result"
```

### Step 4: Verify Test Passes

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/test_custom_tool.py::test_custom_tool_execution -v
# Expected: PASSED
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/test_custom_tool.py::test_custom_tool_execution -v
# Expected: PASSED
```

### Step 5: Run All Tests

**Linux/macOS:**
```bash
source .venv/bin/activate
pytest tests/ -v
# Expected: All tests pass
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pytest tests/ -v
# Expected: All tests pass
```

### Step 6: Run Linter

**Linux/macOS:**
```bash
source .venv/bin/activate
pre-commit run -a
# Expected: All checks pass
```

**Windows:**
```powershell
.venv\Scripts\Activate.ps1
pre-commit run -a
# Expected: All checks pass
```

## Example: Adding New Agent

### Step 1: Write Test
```python
# tests/test_custom_agent.py
@pytest.mark.asyncio
async def test_custom_agent_creation():
    """Test CustomAgent creation."""
    agent_def = AgentDefinition(
        name="custom_agent",
        base_class=CustomAgent,
        tools=[ReasoningTool],
        ...
    )
    agent = await AgentFactory.create(agent_def, task_messages=[...])
    assert isinstance(agent, CustomAgent)
```

### Step 2: Implement Agent (Layer 4)
```python
# sgr_agent_core/agents/custom_agent.py
class CustomAgent(BaseAgent):
    """Custom agent implementation."""
    name: str = "custom_agent"

    async def _reasoning_phase(self) -> ReasoningTool:
        """Reasoning phase implementation."""
        ...

    async def _select_action_phase(self, reasoning: ReasoningTool) -> BaseTool:
        """Select action phase implementation."""
        ...

    async def _action_phase(self, tool: BaseTool) -> str:
        """Action phase implementation."""
        ...
```

## Forbidden

❌ Implement multiple classes simultaneously
❌ Move to upper layers before lower ones are ready
❌ Skip writing tests
❌ Skip running tests
❌ Skip running linter
❌ Implement functionality without understanding architecture

## Allowed

✅ Implement one class at a time
✅ Write tests first (TDD)
✅ Verify test fails before implementation
✅ Verify test passes after implementation
✅ Run all tests after each change
✅ Run linter after each change
✅ Use stubs for dependencies
✅ Refactor after basic functionality works

## References

@architecture.mdc
@code-style.mdc
@testing.mdc
@workflow.mdc

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
