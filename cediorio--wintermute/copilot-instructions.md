## wintermute

> This document outlines the test-driven development approach for building Wintermute.

# Wintermute Development Plan

This document outlines the test-driven development approach for building Wintermute.

## Development Phases

### Phase 1: Foundation (Models & Core Services)
**Goal**: Establish core data models and service interfaces

#### 1.1 Data Models
**Tests to Write**:
- `tests/test_models/test_persona.py`
  - ✓ Persona creation with valid data
  - ✓ Persona validation (required fields)
  - ✓ Persona serialization to/from JSON
  - ✓ Default values for optional fields
  
- `tests/test_models/test_message.py`
  - ✓ Message creation (user/assistant roles)
  - ✓ Message timestamp generation
  - ✓ Message formatting for display
  - ✓ Message metadata handling

**Implementation**:
- Create `Persona` class with fields: id, name, description, system_prompt, temperature, traits
- Create `Message` class with fields: role, content, timestamp, metadata
- Add validation using Pydantic
- Implement JSON serialization

#### 1.2 Configuration Management
**Tests to Write**:
- `tests/test_utils/test_config.py`
  - ✓ Load configuration from .env
  - ✓ Validate required settings
  - ✓ Default values for optional settings
  - ✓ Override with environment variables

**Implementation**:
- Create `Config` class using pydantic-settings
- Support .env file loading
- Validate URLs and API keys
- Provide sensible defaults

#### 1.3 Ollama Client
**Tests to Write**:
- `tests/test_services/test_ollama_client.py`
  - ✓ Initialize client with config
  - ✓ Check connection status
  - ✓ Generate response from prompt
  - ✓ Handle streaming responses
  - ✓ Error handling (connection failed, timeout)
  - ✓ Custom parameters (temperature, etc.)

**Implementation**:
- Create `OllamaClient` class
- Use httpx for async HTTP requests
- Implement `generate()` method
- Implement `stream()` method for real-time responses
- Add connection health check
- Handle errors gracefully

#### 1.4 OpenMemory Client
**Tests to Write**:
- `tests/test_services/test_memory_client.py`
  - ✓ Initialize client with config
  - ✓ Store memory item
  - ✓ Query memories by text
  - ✓ Retrieve user summary
  - ✓ Delete specific memories
  - ✓ Get memory stats
  - ✓ Error handling (API down, invalid key)

**Implementation**:
- Create `MemoryClient` class
- Implement CRUD operations for memories
- Add query method with filters
- Support user isolation (user_id)
- Implement connection health check
- Cache frequently accessed data

#### 1.5 Persona Manager
**Tests to Write**:
- `tests/test_services/test_persona_manager.py`
  - ✓ Load personas from directory
  - ✓ Get persona by ID
  - ✓ List all personas
  - ✓ Switch active persona
  - ✓ Validate persona data
  - ✓ Handle missing persona files
  - ✓ Reload personas dynamically

**Implementation**:
- Create `PersonaManager` class
- Load JSON persona files
- Validate persona schemas
- Track active persona
- Support hot-reloading

### Phase 2: UI Components
**Goal**: Build individual UI components with Textual

#### 2.1 Status Pane
**Tests to Write**:
- `tests/test_ui/test_status_pane.py`
  - ✓ Render initial status
  - ✓ Update connection status
  - ✓ Display memory count
  - ✓ Show model information
  - ✓ Handle status changes
  - ✓ Color coding (connected=green, disconnected=red)

**Implementation**:
- Create `StatusPane` widget (Static)
- Display connection indicators
- Show real-time stats
- Auto-refresh every 5 seconds
- Use Rich markup for styling

#### 2.2 Persona Pane
**Tests to Write**:
- `tests/test_ui/test_persona_pane.py`
  - ✓ Render persona list
  - ✓ Highlight active persona
  - ✓ Handle persona selection
  - ✓ Display persona descriptions on hover
  - ✓ Keyboard navigation
  - ✓ Update when personas change

**Implementation**:
- Create `PersonaPane` widget (ListView)
- Display all available personas
- Highlight current selection
- Emit events on selection change
- Add tooltips for descriptions
- Support keyboard shortcuts (1-9 for quick select)

#### 2.3 Chat Pane
**Tests to Write**:
- `tests/test_ui/test_chat_pane.py`
  - ✓ Render message history
  - ✓ Display user messages
  - ✓ Display assistant messages
  - ✓ Auto-scroll to bottom
  - ✓ Handle input submission
  - ✓ Show typing indicator
  - ✓ Handle long messages (wrapping)
  - ✓ Timestamp display

**Implementation**:
- Create `ChatPane` widget (Container)
- Use `VerticalScroll` for message history
- Add `Input` widget for user input
- Format messages with Rich styling
- Implement auto-scrolling
- Add typing indicator animation
- Handle markdown in messages

### Phase 3: Integration
**Goal**: Connect all components into working application

#### 3.1 Main Application
**Tests to Write**:
- `tests/test_app.py`
  - ✓ App initialization
  - ✓ Layout composition
  - ✓ Service initialization
  - ✓ Event routing
  - ✓ Graceful shutdown
  - ✓ Error recovery

**Implementation**:
- Create main `WintermuteApp` class
- Compose layout with Grid
- Initialize all services
- Set up event handlers
- Implement message loop:
  1. User input → Ollama
  2. Retrieve relevant memories
  3. Generate response with context
  4. Store new memories
  5. Display response
- Add error boundaries
- Implement graceful shutdown

#### 3.2 Message Flow
**Tests to Write**:
- `tests/test_integration/test_message_flow.py`
  - ✓ End-to-end message handling
  - ✓ Memory retrieval before response
  - ✓ Memory storage after response
  - ✓ Persona context injection
  - ✓ Error propagation
  - ✓ Retry logic

**Implementation**:
- Create `MessageHandler` class
- Coordinate between services
- Build context from memories
- Format prompts with persona + context
- Handle streaming responses
- Store conversation in OpenMemory
- Update UI in real-time

### Phase 4: Polish & Advanced Features
**Goal**: Add quality-of-life features and optimizations

#### 4.1 Features
- **Memory Visualization**
  - Show memory relevance scores
  - Display memory timeline
  - Allow memory inspection/editing

- **Conversation History**
  - Save/load conversations
  - Export to markdown
  - Search history

- **Advanced Persona Features**
  - Persona creation wizard
  - Dynamic trait adjustment
  - Persona comparison

- **Performance**
  - Message caching
  - Lazy loading
  - Background memory updates

#### 4.2 Testing
- Integration tests
- Performance tests
- UI snapshot tests
- Load testing

## TDD Principles

### Red-Green-Refactor Cycle

1. **Red**: Write a failing test
   ```python
   def test_persona_creation():
       persona = Persona(
           id="test",
           name="Test Persona",
           system_prompt="You are helpful"
       )
       assert persona.id == "test"
       assert persona.temperature == 0.7  # default value
   ```

2. **Green**: Write minimal code to pass
   ```python
   class Persona:
       def __init__(self, id: str, name: str, system_prompt: str):
           self.id = id
           self.name = name
           self.system_prompt = system_prompt
           self.temperature = 0.7
   ```

3. **Refactor**: Improve while keeping tests green
   ```python
   from pydantic import BaseModel, Field
   
   class Persona(BaseModel):
       id: str
       name: str
       system_prompt: str
       temperature: float = Field(default=0.7, ge=0.0, le=2.0)
   ```

### Testing Best Practices

1. **Test Naming**: `test_<feature>_<scenario>_<expected_result>`
   - `test_persona_creation_with_valid_data_succeeds`
   - `test_persona_creation_missing_name_raises_error`

2. **Test Structure** (Arrange-Act-Assert):
   ```python
   def test_ollama_generate_with_prompt_returns_response():
       # Arrange
       client = OllamaClient(config)
       prompt = "Hello, world!"
       
       # Act
       response = await client.generate(prompt)
       
       # Assert
       assert response is not None
       assert len(response) > 0
   ```

3. **Fixtures**: Use pytest fixtures for reusable setup
   ```python
   @pytest.fixture
   def mock_ollama_client():
       return AsyncMock(spec=OllamaClient)
   
   @pytest.fixture
   def sample_persona():
       return Persona(
           id="test",
           name="Test",
           system_prompt="Test prompt"
       )
   ```

4. **Mocking External Services**:
   ```python
   @pytest.mark.asyncio
   async def test_memory_client_store(mock_httpx):
       mock_httpx.post.return_value = Response(200, json={"id": "123"})
       client = MemoryClient(config)
       
       memory_id = await client.store("test content", user_id="user1")
       
       assert memory_id == "123"
       mock_httpx.post.assert_called_once()
   ```

## Development Workflow

### Sprint Planning

**Sprint 1** (Week 1): Phase 1.1-1.2 (Models & Config)
- Set up project structure
- Write tests for models
- Implement data models
- Configuration management

**Sprint 2** (Week 2): Phase 1.3-1.5 (Services)
- Ollama client
- OpenMemory client
- Persona manager

**Sprint 3** (Week 3): Phase 2 (UI Components)
- Status pane
- Persona pane
- Chat pane

**Sprint 4** (Week 4): Phase 3 (Integration)
- Main app assembly
- Message flow
- End-to-end testing

**Sprint 5** (Week 5): Phase 4 (Polish)
- Advanced features
- Performance optimization
- Documentation

### Daily TDD Routine

1. Review previous day's work
2. Pick next test from plan
3. Write failing test
4. Implement feature
5. Refactor
6. Commit with message: "feat: <feature> (TDD)"
7. Repeat

### Definition of Done

Feature is complete when:
- [ ] All tests pass
- [ ] Code coverage >80%
- [ ] No linting errors
- [ ] Documentation updated
- [ ] Manual testing performed
- [ ] Peer review completed (if applicable)

## Testing Strategy

### Unit Tests
- Test individual functions/methods
- Mock external dependencies
- Fast execution (<1s per test)
- High coverage (>90%)

### Integration Tests
- Test service interactions
- Use real services in containers
- Moderate speed (~5s per test)
- Critical paths covered

### UI Tests
- Test widget rendering
- Mock service responses
- Use Textual's testing utilities
- Cover user interactions

### End-to-End Tests
- Full application flow
- Real services (dev environment)
- Slower execution (~30s per test)
- Happy paths and error scenarios

## Tools & Libraries

### Core Dependencies
```toml
[project]
dependencies = [
    "textual>=0.40.0",
    "httpx>=0.25.0",
    "pydantic>=2.4.0",
    "pydantic-settings>=2.0.0",
    "python-dotenv>=1.0.0",
    "rich>=13.6.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.21.0",
    "pytest-cov>=4.1.0",
    "pytest-watch>=4.2.0",
    "pytest-mock>=3.11.1",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
]
```

### Development Commands
```bash
# Install dependencies
uv pip install -e ".[dev]"

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=wintermute --cov-report=html

# Watch mode
uv run pytest-watch

# Linting
uv run ruff check .
uv run ruff format .

# Type checking
uv run mypy src/

# Run app
uv run python -m wintermute.app
```

## Success Metrics

### Code Quality
- Test coverage: >80%
- Type coverage: >90%
- No critical security issues
- Maintainability index: A

### Performance
- Message latency: <2s
- UI responsiveness: 60 FPS
- Memory usage: <100MB
- Startup time: <1s

### User Experience
- Keyboard navigation works
- No UI freezes
- Clear error messages
- Intuitive persona switching

## Future Enhancements

1. **Multi-user support**
   - User authentication
   - Separate memory spaces
   - User profiles

2. **Advanced memory features**
   - Memory importance scoring
   - Memory decay simulation
   - Memory visualization graph

3. **Plugin system**
   - Custom personas via plugins
   - Third-party LLM providers
   - Custom UI themes

4. **Export/Import**
   - Export conversations
   - Import persona packs
   - Backup/restore memories

5. **Analytics**
   - Conversation insights
   - Memory usage patterns
   - Persona effectiveness

## Resources

- [Textual Documentation](https://textual.textualize.io/)
- [Ollama API Docs](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [OpenMemory API](https://openmemory.cavira.app/)
- [Pytest Best Practices](https://docs.pytest.org/en/stable/goodpractices.html)
- [TDD by Example](https://www.oreilly.com/library/view/test-driven-development/0321146530/)
- [uv Documentation](https://github.com/astral-sh/uv)

---
> Source: [cediorio/wintermute](https://github.com/cediorio/wintermute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
