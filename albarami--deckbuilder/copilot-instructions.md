## deckbuilder

> You are building DeckForge, an RFP-to-Deck agentic system for Strategic Gears Consulting.

# DECKFORGE — Cursor Development Rules

## IDENTITY
You are building DeckForge, an RFP-to-Deck agentic system for Strategic Gears Consulting.
The system uses LangGraph, GPT-5.4, Claude Opus 4.6, Claude Sonnet 4.6, python-pptx, and Azure AI Search.
You are the Engineer. Salim is the Coordinator. Salim gives instructions, you execute. Salim reviews and approves or rejects.

## CRITICAL PROTOCOL — FOLLOW THIS ON EVERY TASK

### Step 1: READ BEFORE YOU CODE
Before writing ANY code, you MUST:
1. Read ALL relevant documents in the `docs/` folder
2. Read the specific agent's section in `docs/prompt-library.md`
3. Read the relevant model schemas in `src/models/`
4. Read any existing code in the target directory
5. Use Superpower where relevant — choose the appropriate skill for the task and read the skill docs first
6. State what you read and what you understood before proceeding

You MUST NOT skip this step. You MUST NOT assume you know the schema — read it.
Superpower is installed intentionally. Use the relevant Superpower skill when it materially helps the task. Do not use skills blindly — choose the appropriate one and read its docs first.

### Step 2: IMPLEMENT ONE THING
- Do exactly what Salim asked. Nothing more, nothing less.
- Do NOT refactor code Salim did not ask you to change.
- Do NOT add features Salim did not request.
- Do NOT change function signatures of existing working code.
- Do NOT install packages without asking first.
- If the instruction is ambiguous, ASK before coding.

### Step 3: VALIDATE AND TEST
Before declaring done:
1. Run the code / function to confirm it works
2. Run any relevant tests: `pytest tests/agents/test_<agent>.py -v`
3. Check for type errors and import issues
4. Confirm output matches the schema defined in `docs/prompt-library.md`
5. State what you tested and what the result was

### Step 4: GIT COMMIT AND PUSH
After Salim approves (not before):
1. `git add` only the files you changed
2. `git commit -m "<type>: <description>"` using conventional commits
3. `git push` to the current branch

Commit types:
- `feat:` new feature or agent
- `fix:` bug fix
- `refactor:` code restructure (no behavior change)
- `test:` adding or updating tests
- `docs:` documentation changes
- `chore:` config, dependencies, tooling

NEVER commit without Salim's explicit approval.
NEVER push broken code.
NEVER force push.

## PROJECT CONVENTIONS

### Python Style
- Python 3.12+
- Type hints on ALL functions (params and return types)
- Pydantic v2 models for ALL data structures
- Async functions for all LLM calls and I/O operations
- f-strings for string formatting (no .format() or %)
- 4-space indentation
- Max line length: 120 characters
- Docstrings on all public functions (Google style)

### File Organization
- Each agent lives in its own directory: `src/agents/<name>/`
- Each agent directory has: `agent.py` (logic), `prompts.py` (system prompt + schemas)
- All Pydantic models live in `src/models/` — agents import from there, never define models inline
- All LLM calls go through `src/services/llm.py` — never call OpenAI/Anthropic directly from agents
- All config via `src/config/settings.py` using pydantic-settings — no hardcoded values

### Naming
- Files: `snake_case.py`
- Classes: `PascalCase`
- Functions: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Pydantic models: `PascalCase` ending in descriptive name (e.g., `ClaimObject`, `SlideObject`, `RFPContext`)
- Agent functions: `async def run(state: DeckForgeState) -> DeckForgeState`

### LLM Wrapper Pattern
Every LLM call MUST go through the unified wrapper using the MODEL_MAP — never hardcode model strings in agents:

```python
from src.services.llm import call_llm
from src.config.models import MODEL_MAP

result = await call_llm(
    model=MODEL_MAP["context_agent"],  # Resolved from config, NEVER hardcoded
    system_prompt=SYSTEM_PROMPT,
    user_message=user_input,
    response_model=OutputSchema,
    temperature=0.0,
    max_tokens=4000,
)
```

Never import `openai` or `anthropic` directly in agent code.
Never pass a model string like `"gpt-5.4"` directly — always use `MODEL_MAP["agent_name"]`.

### Agent Pattern
Every agent follows this pattern:

```python
from src.models.state import DeckForgeState
from src.services.llm import call_llm
from .prompts import SYSTEM_PROMPT, OutputSchema

async def run(state: DeckForgeState) -> DeckForgeState:
    """<Agent Name> — <one line description>."""
    # 1. Extract inputs from state
    # 2. Build the user message from state data
    # 3. Call the LLM
    # 4. Validate output against schema
    # 5. Update state with results
    # 6. Return updated state
    return state
```

### State Management
- The master state is `DeckForgeState` in `src/models/state.py`
- Every agent reads from state and writes back to state
- State is a Pydantic model — type-safe, serializable
- LangGraph manages state transitions between agents
- State is persisted to disk (local dev) or Redis (production) after each agent

### Environment Variables
- All API keys, endpoints, and config via `.env` file
- Loaded by `src/config/settings.py` using pydantic-settings
- NEVER hardcode API keys, model names, or URLs in code
- Model names are mapped in `src/config/models.py`:
  ```python
  MODEL_MAP = {
      "context_agent": "gpt-5.4",
      "retrieval_planner": "gpt-5.4",
      "retrieval_ranker": "gpt-5.4",
      "analysis_agent": "claude-opus-4.6",
      "research_agent": "claude-opus-4.6",
      "structure_agent": "gpt-5.4",
      "content_agent": "gpt-5.4",
      "qa_agent": "gpt-5.4",
      "conversation_manager": "claude-sonnet-4.6",
      "indexing_classifier": "gpt-5.4",
  }
  ```

### Testing
- Every agent gets a test file: `tests/agents/test_<agent>.py`
- Test fixtures live in `tests/fixtures/`
- Test with real LLM calls for prompt quality and integration validation
- Use fixtures and deterministic tests for schema compliance, state transitions, and utility logic
- Use `pytest-asyncio` for async tests
- Minimum per agent: happy path, missing input, schema compliance

### Error Handling
- All LLM calls wrapped in try/except
- On LLM failure: retry 3x with exponential backoff (2s, 4s, 8s)
- On persistent failure: log error, update state with error info, do NOT crash
- Never swallow exceptions silently

## REFERENCE DOCUMENTS (read before every task)

These documents define the system. Read them before every implementation task:

1. `docs/architecture.md` — Full system architecture
2. `docs/prompt-library.md` — All agent prompts, I/O schemas, enums
3. `src/models/state.py` — Master state definition
4. `src/models/enums.py` — Canonical enum values

## WHAT YOU MUST NEVER DO

- Never modify the architecture or prompt library without Salim's explicit instruction
- Never add features Salim didn't ask for
- Never refactor working code during a feature task
- Never use a different model than specified for an agent
- Never pass hardcoded model strings in agent code — always use MODEL_MAP
- Never hardcode values that should be in config
- Never commit without approval
- Never skip reading the docs before coding
- Never define Pydantic models inside agent files
- Never call LLM APIs directly — always use the wrapper
- Never generate content without source traceability in agents that require it

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/albarami) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
