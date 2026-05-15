## langchain-examples

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an agentic system that takes articles from websites and generates short video scripts through a multi-agent pipeline. The system uses LangChain/LangGraph for orchestration with a terminal input/output interface.

**Key agents:**
- **Writer**: Generates video scripts from article content (700-1000 chars), has scraper tool for fetching URLs
- **Editor**: Reviews scripts for tone match, hook quality, clarity, length (can reject)
- **FactChecker**: Verifies factual accuracy against source article (can reject)
- **User**: Human-in-the-loop approval/feedback after each iteration

Agents can reject and return work to the writer with feedback, creating an iterative improvement loop.

## Development Setup

**Install dependencies:**
```bash
uv sync
```

**Run linter:**
```bash
uv run ruff check .
```

**Auto-fix linting issues:**
```bash
uv run ruff check --fix .
```

**Format code:**
```bash
uv run ruff format .
```

## Architecture

### Core Design Pattern: LangGraph State Machine

The system uses **LangGraph** (not plain LCEL) because it provides:
- Built-in cycle support for rejection loops
- Conditional edge routing (editor → writer if rejected, → factchecker if approved)
- Automatic state management
- Native async/await support

**Pipeline flow:**
```
Article → Writer → Editor → FactChecker → User Approval
             ↑________|_________|
            (rejection with feedback)
```

### State Management

Central state flows through all agents as `PipelineState` (TypedDict):

```python
class PipelineState(TypedDict):
    messages: Annotated[list[BaseMessage], add]  # Unified conversation history
    drafts: Annotated[list[str], add]            # All script versions
    article_content: Annotated[list[str], add]   # Source articles (can have multiple)
    iteration: int                                # Rejection loop counter
    editor_approved: bool                         # Editor approval flag
    factchecker_approved: bool                    # FactChecker approval flag
    user_approved: bool                           # User exit signal
```

**Key patterns:**
- **Unified messages**: All conversation (user, writer, editor, factchecker) in one list
- **Message identification**: Each agent uses `AIMessage(content=..., name="writer"|"editor"|"factchecker")`
- **State accumulation**: `Annotated[list, add]` automatically appends to lists
- **Immutability**: Agents return partial state updates, LangGraph merges them

State is immutable between nodes; agents return dict updates that get merged into state.

### Agent Interface

**Current implementation**: Agents are simple functions that take `state` and return partial state updates.

```python
def writer_agent(state: PipelineState) -> dict:
    """Generate script from article."""
    # Build message history with system prompt
    messages = [SystemMessage(content=WRITER_PROMPT)] + state["messages"]

    # Add article context
    if state["article_content"]:
        messages.append(HumanMessage(content=f"Article:\n{article_text}"))

    # Tool calling loop for scraper
    response = writer_llm.invoke(messages)

    # Return updates
    return {
        "messages": [AIMessage(content=response.content, name="writer")],
        "drafts": [script],
        "iteration": state["iteration"] + 1,
    }
```

**Agent patterns:**
- **user_input_node**: Gets terminal input, returns `HumanMessage`
- **writer_agent**: Invokes LLM with tool calling loop, returns `AIMessage` with script
- **editor_agent**: Uses LCEL chain + structured output (Pydantic), returns approval + feedback
- **factchecker_agent**: Same pattern as editor

**Routing functions** (in `routes.py`) determine next node based on approval flags and iteration count.

### Future: Plugin Architecture (Planned)

The system is designed to eventually support:
- Abstract `BaseAgent` class with `@register_agent` decorator
- Config-driven agent composition (`config.yaml`)
- Zero-code agent addition

Currently: Functional implementation for simplicity and rapid iteration.

## Project Structure

```
src/langchain_examples/
├── agents/
│   ├── agents.py     # All agent functions (writer, editor, factchecker, user_input)
│   ├── prompts.py    # System prompts (WRITER_PROMPT, EDITOR_PROMPT, FACTCHECKER_PROMPT)
│   ├── models.py     # Pydantic models (EditorOutput, FactCheckerOutput)
│   ├── routes.py     # Routing functions (route_after_writer, route_after_editor, etc.)
│   └── state.py      # PipelineState TypedDict
├── tools/
│   └── scrapers.py   # Article scraper tool
└── main.py           # Pipeline builder and executor

examples/             # Example scripts
├── example1.py
└── example2.py

docs/                 # Architecture documentation
├── architecture.md
├── design-decisions.md
└── extensibility-guide.md

data/                 # Runtime data
└── runs/
    └── YYYYMMDD_HHMMSS/
        ├── pipeline_result.json
        ├── pipeline_graph.png
        └── final_script.txt

TODO.md               # Feature roadmap
CLAUDE.md             # This file
```

## Key Design Decisions

### Functional Approach (Current)
- **Functions** for: Agents (writer_agent, editor_agent, etc.), routing logic
- **TypedDict** for: State definition (simple, no classes needed)
- **Pydantic** for: LLM structured outputs (EditorOutput, FactCheckerOutput)
- Rationale: Simple, fast iteration, easy to understand and debug

### Unified Message History (Nov 2024)
- Single `messages: list[BaseMessage]` field for all conversation
- Replaces separate `user_input`, `feedback`, and agent message tracking
- Each agent identified by `name` parameter: `AIMessage(content=..., name="writer")`
- Follows LangChain best practices for multi-agent systems

### Model Selection Strategy
- **Current**: All agents use `gpt-5-mini` (placeholder model name)
- **Planned**:
  - Writer: `gpt-4o-mini` with temperature=0.7 (cheap, creative)
  - Editor: `gpt-4o` with temperature=0.3 (better judgment)
  - FactChecker: `gpt-4o` with temperature=0 (deterministic)
  - Cost optimization: ~47% cheaper than using gpt-4o everywhere

### Tone Matching (Nov 2024)
- Writer analyzes user request for tone (funny/serious/ironic/etc.)
- Applies requested tone from FIRST draft, not after editor feedback
- Editor prioritizes tone match as #1 evaluation criterion
- Prevents dry rewrites after factchecker corrections

### Rejection Loop with Message History
- All feedback visible in unified `messages` list
- Writer sees complete conversation context automatically
- Max iterations (5) prevents infinite loops and runaway costs
- Approval flags: `editor_approved`, `factchecker_approved`

### Tool Calling Pattern
- Writer has inline tool-calling loop for scraper tool
- Alternative: Separate tool executor node (not implemented)
- Current approach: simpler, fewer nodes in graph

### Async Strategy
- Phase 1 (current): Synchronous (`invoke()`)
- Phase 2 (future): Async (`ainvoke()`)
- LangGraph makes this transition trivial - just add `async`/`await`

### Persistence
- JSON files for pipeline results (serialized messages)
- PNG graph visualization
- Plain text final script
- Future: SQLite checkpointing for resume/replay

## Important Implementation Notes

### When Writing Agents

- Agents are **functions**: `def agent_name(state: PipelineState) -> dict:`
- Return **partial state updates** as dict (LangGraph merges them)
- Access full state context via `state` parameter
- Return messages with `name` parameter: `AIMessage(content=..., name="agent_name")`
- Use Pydantic models for structured LLM outputs (editor/factchecker)
- Writer can't use structured output (tool binding conflict)

### When Modifying Pipeline

- Pipeline is built in `main.py` using LangGraph's `StateGraph`
- Add nodes: `workflow.add_node("agent_name", agent_function)`
- Add conditional edges with routing functions and dict mappings:
  ```python
  workflow.add_conditional_edges(
      "editor_agent",
      route_after_editor,
      {"approved": "factchecker_agent", "rejected": "writer_agent"}
  )
  ```
- Test changes with graph visualization: `app.get_graph().draw_mermaid_png()`

### State Updates

- State updates happen implicitly through dict returns
- **Never mutate state directly** - always return new values
- Use `Annotated[list, add]` for list accumulation (messages, drafts, article_content)
- If adding new state fields:
  1. Update `PipelineState` TypedDict in `agents/state.py`
  2. Initialize in `main.py` initial state dict
  3. Update relevant agents to handle new field

### Known Issues

- **Approval flag persistence**: Flags don't reset when user provides new feedback
  - If editor approves, then user asks for changes, writer skips editor on next iteration
  - Fix: Reset flags in `user_input_node` on new user input

## Testing Strategy

Run tests with pytest (when available):
```bash
uv run pytest tests/
```

**Test coverage priorities:**
1. Individual agent behavior (unit tests)
2. Pipeline flow with mocked agents (integration tests)
3. Rejection loop mechanics (edge case tests)

## Documentation

Comprehensive docs in `docs/`:
- `architecture.md`: System design and LangGraph implementation
- `design-decisions.md`: Why we made specific choices (OOP vs functional, models, persistence, etc.)
- `extensibility-guide.md`: How to add agents, extend state, custom routing, performance tips

Read these before making architectural changes.

## Tech Stack Constraints

**Keep it simple and cheap:**
- Terminal UI only (Textual), no web interface
- No message brokers (Redis, RabbitMQ)
- No Docker requirement
- Direct OpenAI API calls
- File-based or SQLite storage only

Don't introduce complex infrastructure unless absolutely necessary.

---
> Source: [kkruglik/langchain_examples](https://github.com/kkruglik/langchain_examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
