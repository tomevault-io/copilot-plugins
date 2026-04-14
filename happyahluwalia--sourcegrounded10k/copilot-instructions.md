## sourcegrounded10k

> You are an Expert AI Developer.


You are an Expert AI Developer.
Below are guidelines however if there are best practices that you are aware off that are different than below guidelines share it with me with all pros and cons. I will then make a decision.
KEY IS SIMPLICITY. We need to focus on features we can ship fast. We can add additional things incrementally.
Always keep documentation updated. Keep adding different tradeoffs we make and why
Review docs/Data_Safety_Rules.md to avoid critical errors
## Core Development Principles

### 1. Agent Design Principles

```
AGENT DESIGN RULES:
- Each agent should have a single, clear responsibility
- Agents communicate only through shared state (no side effects)
- Always include type hints for agent functions
- Agent prompts should be stored in separate files (prompts/ directory)
- Never hardcode prompts in Python code
- Each agent should be testable in isolation
- Use async/await for all agent operations
- Log every agent decision and action


### 2. State Management Rules

```
STATE MANAGEMENT RULES:
- Always use TypedDict for state schemas
- Use Annotated with reducer functions for accumulated fields
- Never mutate state in place (return new state)
- Include metadata fields: timestamp, execution_log, errors
- Keep state flat when possible (avoid deep nesting)
- Always provide default values in state schema
- Document what each state field represents
- Use Langraph checkpoint


### 3. Error Handling and Resilience

ERROR HANDLING RULES:
- Wrap all tool calls in try-except blocks
- Implement exponential backoff for retries
- Log errors with full context (state, input, stack trace)
- Provide graceful degradation (partial results better than nothing)
- Never let exceptions crash the entire graph
- Use error states to trigger retry or fallback logic
- Always inform user when partial results returned

### 4. LangGraph Specific Rules

LANGGRAPH RULES:
- Always use StateGraph, not MessageGraph
- Compile graph with checkpointer for production
- Use conditional edges for dynamic routing
- Test graphs with simple inputs before complex ones
- Visualize graphs using mermaid diagrams in docs
- Name nodes with clear, descriptive names
- Use START and END constants, never strings
- Always provide RunnableConfig for thread management


### 5. Prompt Engineering Rules


PROMPT ENGINEERING RULES:
- Store prompts in separate .txt or .md files in prompts/ directory
- Use Jinja2 templates for dynamic prompt generation
- Include clear instructions, examples, and output format
- Specify what the agent should NOT do (negative examples)
- Use XML tags for structured output parsing
- Test prompts independently before integrating
- Version control prompts alongside code
- Document what each prompt template variable expects

Prompt file structure:

prompts/
├── lead_agent_prompt.txt
├── rag_agent_prompt.txt
├── web_search_prompt.txt
├── analysis_prompt.txt
└── citation_prompt.txt


### 6. Tool Development Rules


TOOL DEVELOPMENT RULES:
- Use @tool decorator from langchain.tools
- Provide comprehensive docstrings (used by LLM to decide tool usage)
- Include type hints for all parameters
- Return structured data (Dict), not strings
- Handle errors internally (don't let exceptions escape)
- Add usage examples in docstring
- Specify when to use vs when NOT to use
- Test tools independently before agent integration


### 7. Testing Rules

TESTING RULES:
- Write tests for each agent independently
- Mock LLM calls in unit tests (use FakeLLM)
- Test both success and failure paths
- Test state transitions explicitly
- Use pytest fixtures for common state setups
- Test with real LLM in integration tests
- Measure token usage in tests
- Test graph execution end-to-end


### 8. Logging and Observability Rules

LOGGING RULES:
- Use structured logging (JSON format)
- Log at appropriate levels (DEBUG, INFO, WARNING, ERROR)
- Include context: user_id, session_id, agent_name, step
- Log token usage for cost tracking
- Log latency for each agent and tool call
- Never log sensitive user data (PII)
- Use correlation IDs to trace requests across agents
- Log to stdout for containerized environments


### DECISION LOGGING RULES:
- Log every agent decision with options considered
- Capture reasoning for each choice
- Track confidence scores
- Build decision tree structure for UI visualization
- Stream thinking process in real-time

### 9. Performance Optimization Rules

PERFORMANCE RULES:
- Use async/await for all I/O operations
- Implement parallel execution where possible (multiple agents)
- Cache expensive operations (embeddings, LLM calls for identical inputs)
- Use connection pooling for databases
- Implement request batching for vector DB queries
- Monitor and optimize token usage
- Use smaller models for simple tasks (Llama 3.1 8B vs 70B)
- Implement timeouts for all external calls

### 10. Code Organization Rules

```
CODE ORGANIZATION RULES:
- Follow this directory structure:
  
  finance_agent/
  ├── agents/
  │   ├── __init__.py
  │   ├── lead_agent.py
  │   ├── rag_agent.py
  │   ├── web_search_agent.py
  │   ├── analysis_agent.py
  │   └── citation_agent.py
  ├── graphs/
  │   ├── __init__.py
  │   ├── finance_graph.py
  │   └── graph_builder.py
  ├── tools/
  │   ├── __init__.py
  │   ├── sec_search.py
  │   ├── web_search.py
  │   ├── calculator.py
  │   └── code_executor.py
  ├── prompts/
  │   ├── lead_agent_prompt.txt
  │   ├── rag_agent_prompt.txt
  │   └── ...
  ├── schemas/
  │   ├── __init__.py
  │   ├── state.py
  │   └── models.py
  ├── services/
  │   ├── __init__.py
  │   ├── qdrant_service.py
  │   ├── ollama_service.py
  │   └── cache_service.py
  ├── utils/
  │   ├── __init__.py
  │   ├── logging.py
  │   ├── metrics.py
  │   └── validators.py
  └── tests/
      ├── unit/
      ├── integration/
      └── fixtures/

- One agent per file
- One tool per file
- Keep prompts separate from code
- Use dependency injection for services
- Follow PEP 8 style guide
- Use type hints everywhere
- Maximum function length: 50 lines
```
## Configuration Management


CONFIGURATION RULES:
- Use environment variables for all secrets
- Use .env files for local development (never commit)
- Use pydantic BaseSettings for config validation
- Separate config for dev/staging/prod
- Never hardcode URLs, API keys, or model names

---
## EVALUATION RULES:
- Start with 20 test queries representing real usage
- Use LLM-as-judge for grading (factual accuracy, citations, completeness)
- Human testing for edge cases and emergent behaviors
- Track: success rate, tool efficiency, source quality, latency
- Test both outcomes AND process (did agent follow reasonable path?)
---

---

## Documentation Rules

DOCUMENTATION RULES:
- Every agent needs a docstring explaining its purpose
- Every tool needs usage examples in docstring
- Document state transitions in graph builder
- Keep a CHANGELOG.md for agent behavior changes
- Document prompt template variables
- Add inline comments for complex logic only
- Use type hints as self-documenting code
- Maintain architecture diagrams in docs/

## Security Rules

SECURITY RULES:
- Never log API keys or tokens
- Validate all user inputs (SQL injection, XSS prevention)
- Use parameterized queries for database
- Sanitize file paths in file operations
- Implement rate limiting on API endpoints
- Use HTTPS for all external calls
- Implement authentication for multi-agent API
- Sandbox code execution (if using code execution tool)


## Deployment and Operations Rules

DEPLOYMENT RULES:
- Use Docker for all services
- Pin all dependency versions in requirements.txt
- Use multi-stage Docker builds for smaller images
- Implement health check endpoints
- Use environment-based configuration
- Implement graceful shutdown
- Use process managers (gunicorn, uvicorn workers)
- Implement circuit breakers for external services
- Incremental releases should NEVER break current deployment
---

## Common Pitfalls to Avoid

1. **Don't** hardcode prompts in Python code → Use template files
2. **Don't** use synchronous calls in async functions → Use async/await
3. **Don't** mutate state objects → Return new state
4. **Don't** catch exceptions without logging → Always log errors
5. **Don't** skip type hints → They prevent bugs
6. **Don't** create circular dependencies between agents → Use state for communication
7. **Don't** forget to test with real LLMs → Unit tests with mocks aren't enough
8. **Don't** skip error handling in tools → Tools should never crash agents
9. **Don't** log sensitive data → PII, API keys, user queries (be careful)
10. **Don't** ignore performance metrics → Monitor token usage and latency

---

## Development Workflow
ALWAYS use `ChatOllama` from LangChain for ALL LLM calls.**

**NEVER use direct `ollama.Client` from the Ollama SDK.

1. **Design**: Document agent responsibility and interface
2. **Prompt**: Write and test prompt template independently
3. **Implement**: Code agent function with error handling
4. **Test Unit**: Test with mocked LLM and dependencies
5. **Test Integration**: Test with real LLM and services
6. **Add to Graph**: Integrate into graph with proper edges
7. **Test E2E**: Test full graph execution
8. **Document**: Update docs and add examples
9. **Deploy**: Build Docker image and deploy
10. **Monitor**: Watch logs and metrics
11. **Prefer Determinstic call over LLM Call
---
## 🛡️ DATA SAFETY - CRITICAL RULES

### ABSOLUTE PROHIBITIONS

**NEVER recommend these commands without explicit user request:**
- `docker system prune -a` (destroys all images, volumes, data)
- `docker system prune --volumes` (destroys all volumes)
- `docker volume prune` or `docker volume rm` (destroys data)
- `docker-compose down -v` (destroys volumes)
- `rm -rf data/` or any recursive deletes
- `DROP DATABASE` or `TRUNCATE TABLE`

### REQUIRED PROCESS for ANY Destructive Command

1. **Check**: Does this affect data/volumes/databases?
2. **Alternative**: Provide safe alternative first
3. **Warning**: Explicit warning with consequences
4. **Backup**: Provide backup commands
5. **Confirm**: Require user confirmation
6. **Recovery**: Provide recovery steps

### SAFE Docker Cleanup Commands

```bash
# ✅ SAFE - Only removes stopped containers
docker container prune

# ✅ SAFE - Only removes unused images
docker image prune

# ✅ SAFE - Only removes build cache
docker builder prune
```

### GOLDEN RULE

**"When in doubt, DON'T recommend it. Ask the user first."**

If a command deletes/removes/drops anything → STOP. WARN. CONFIRM. BACKUP.

---

**Incident**: Nov 2, 2025 - Recommended `docker system prune -a` which destroyed production PostgreSQL, Qdrant, and Ollama data. Site completely down. Never again.

**Full Documentation**: See DATA_SAFETY_RULES.md
--------------

## Resources and Learning

- LangGraph Docs: https://langchain-ai.github.io/langgraph/
- Anthropic Multi-Agent Guide: https://www.anthropic.com/engineering/multi-agent-research-system
- FastAPI Best Practices: https://fastapi.tiangolo.com/
- Python Async Programming: https://docs.python.org/3/library/asyncio.html
- Structured Logging: https://www.structlog.org/
- Langgraph Checkpoint: https://reference.langchain.com/python/langgraph/checkpoints/#langgraph.checkpoint.postgres.PostgresSaver.list
---

## Final Checklist Before Committing Code

- [ ] All functions have type hints
- [ ] All functions have docstrings
- [ ] Tests pass (pytest)
- [ ] Code formatted (black, isort)
- [ ] No sensitive data in code
- [ ] Error handling implemented
- [ ] Logging added
- [ ] Documentation updated
- [ ] Prompts in separate files
- [ ] Performance considered (async where needed)

Always lookup the documentation if you have any doubt. 
All documents are at @docs folder

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/happyahluwalia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
