## cogentx2

> ai agent creator ruls

# AI Agent Creator Rules and Best Practices

## Core Principles

1. **Type Safety and Validation**
   - Leverage Pydantic for robust data validation
   - Ensure type-safe dependency injection
   - Validate agent responses using structured schemas
   - Use strong typing for all agent interfaces

2. **Agent Architecture**
   - Design agents with single responsibility principle
   - Implement clear input/output contracts
   - Support asynchronous operations by default
   - Enable streaming responses for real-time processing

3. **Multi-Agent System Design**
   - Avoid unnecessary agent dependencies
   - Keep agents stateless when possible
   - Use orchestrator patterns for complex workflows
   - Implement clear communication protocols between agents

## Implementation Guidelines

### Agent Structure
```python
# Example agent structure
from pydantic_ai import Agent, BaseModel
from typing import List, Optional

class AgentInput(BaseModel):
    query: str
    context: Optional[dict] = None

class AgentOutput(BaseModel):
    response: str
    confidence: float
    metadata: dict

class CustomAgent(Agent):
    async def run(self, input: AgentInput) -> AgentOutput:
        # Agent implementation
        pass
```

### Best Practices

1. **Agent Creation**
   - Define clear input/output models using Pydantic
   - Implement error handling and retry mechanisms
   - Add logging and monitoring capabilities
   - Support graceful degradation

2. **Agent Communication**
   - Use typed message passing
   - Implement retry and timeout mechanisms
   - Handle partial failures gracefully
   - Support both sync and async communication

3. **State Management**
   - Use immutable state when possible
   - Implement clear state transitions
   - Support state persistence if needed
   - Handle concurrent state access safely

4. **Error Handling**
   - Define custom error types
   - Implement fallback mechanisms
   - Log errors with context
   - Support graceful degradation

## Production Considerations

1. **Performance**
   - Implement caching where appropriate
   - Use connection pooling
   - Support batched operations
   - Monitor resource usage

2. **Scalability**
   - Design for horizontal scaling
   - Implement load balancing
   - Support distributed deployment
   - Use message queues for async operations

3. **Monitoring**
   - Implement comprehensive logging
   - Add performance metrics
   - Track agent success rates
   - Monitor resource usage

4. **Security**
   - Implement rate limiting
   - Validate all inputs
   - Use secure communication
   - Handle sensitive data properly

## Testing Guidelines

1. **Unit Testing**
   - Test individual agent logic
   - Mock external dependencies
   - Test error conditions
   - Validate output schemas

2. **Integration Testing**
   - Test agent interactions
   - Verify workflow execution
   - Test failure scenarios
   - Validate end-to-end flows

3. **Performance Testing**
   - Test under load
   - Measure response times
   - Verify resource usage
   - Test concurrent operations

## Documentation Requirements

1. **Agent Documentation**
   - Document input/output schemas
   - Describe agent purpose
   - List dependencies
   - Provide usage examples

2. **System Documentation**
   - Document system architecture
   - Describe deployment process
   - List configuration options
   - Provide troubleshooting guides

## Development Workflow

1. **Version Control**
   - Use semantic versioning
   - Maintain changelog
   - Document breaking changes
   - Tag releases properly

2. **Code Quality**
   - Follow PEP 8 guidelines
   - Use type hints
   - Write clear docstrings
   - Maintain test coverage

3. **Dependency Management**
   - Pin dependency versions
   - Use virtual environments
   - Document requirements
   - Regular dependency updates

## Example Multi-Agent Workflow

```python
from pydantic_ai import Agent, Workflow
from typing import List

class TriageAgent(Agent):
    """Routes requests to appropriate specialized agents"""
    pass

class SpecialistAgent(Agent):
    """Handles specific domain tasks"""
    pass

class ReviewAgent(Agent):
    """Reviews and validates agent outputs"""
    pass

# Workflow definition
workflow = Workflow()
workflow.add_agent("triage", TriageAgent())
workflow.add_agent("specialist", SpecialistAgent())
workflow.add_agent("review", ReviewAgent())

# Define workflow steps
workflow.add_step("triage", "specialist")
workflow.add_step("specialist", "review")
```

Remember to always follow these guidelines when creating and maintaining AI agents. Regular review and updates of these practices ensure system reliability and maintainability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/varunisrani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/varunisrani)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
