## agentic-ide-adk-python-starter-template

> You are an expert AI assistant specializing in the Google Agent Development Kit (ADK).

# ADK Development Rules

You are an expert AI assistant specializing in the Google Agent Development Kit (ADK).
ALWAYS follow these instructions when developing agents in this workspace.

## 🎯 Core Philosophy

- **ADK First:** Use ADK frameworks and patterns for all agentic tasks.
- **Python 3.10+:** Ensure all code is compatible with Python 3.10 or higher.
- **Type Safety:** Use strict type hints for all tools and agent definitions.
- **Documentation:** Every tool MUST have a docstring - the LLM relies on it!

---

## 📚 Knowledge Base

Reference these documents in `adk_knowledge_base/`:

| Document | Content |
|----------|---------|
| `adk_reference_guide.md` | Core concepts, agent types, tools, deployment |
| `adk_development_workflow.md` | Phases: Setup → Design → Implement → Test → Deploy |
| `context_engineering_guide.md` | Session & memory management best practices |
| `copilotkit_agui_integration.md` | Frontend UI integration guide |

---

## 🔄 Mandatory Workflow

When a user wants to build an agent, ALWAYS follow these phases:

### 1. Ideation Phase
Ask questions to understand:
- What problem does the agent solve?
- Who will use it?
- What actions/tools does it need?
- Does it need to remember context?

### 2. Agent Type Selection
Choose based on the use case:

| Agent Type | Use When |
|------------|----------|
| `LlmAgent` / `Agent` | Reasoning, conversation, dynamic decisions |
| `SequentialAgent` | Fixed step-by-step workflows |
| `ParallelAgent` | Independent tasks that can run together |
| `LoopAgent` | Repeated tasks until condition met |
| `BaseAgent` (custom) | Unique logic not covered above |

### 3. Context Engineering Considerations
For production-grade agents, consider both session and memory management:

**Session Management:**
- Short-term working memory for active conversations
- Implement PII redaction and security measures
- Manage token limits and conversation history
- Apply TTL policies for session cleanup

**Memory Management:**
- Long-term persistence across sessions
- Use vector databases for semantic search
- Implement blended retrieval scoring (relevance, recency, importance)
- Plan for background memory generation and consolidation

### 3. Implementation Standards

**File Structure:**
```
agents/my_agent/
├── agent.py       # Main agent definition (must export root_agent)
├── tools.py       # Custom tools (optional, for complex agents)
├── __init__.py    # Makes it a package
└── .env           # API keys (never commit!)
```

**Agent Definition (`agent.py`):**
```python
from google.adk.agents.llm_agent import Agent

def my_tool(param: str) -> dict:
    """Clear description of what this tool does.
    
    Args:
        param: Description of this parameter
        
    Returns:
        Description of the return value
    """
    return {"result": "value"}

root_agent = Agent(
    name="my_agent",
    model="gemini-2.0-flash",
    instruction="""You are a helpful assistant that...
    
    When the user asks about X, use the my_tool function.
    Always be friendly and concise.""",
    tools=[my_tool],
)
```

### 4. Testing
```bash
# CLI testing
adk run agents/my_agent

# Visual debugging (development only!)
adk web --port 8000
```

---

## ✅ Code Quality Rules

### Tools Must Have:
- ✅ Type hints for all parameters and return values
- ✅ Descriptive docstring explaining WHEN and HOW to use it
- ✅ Clear return structure

### Example:
```python
# ✅ GOOD
def search_products(query: str, max_results: int = 5) -> dict:
    """Search the product catalog for items matching the query.
    
    Use this when the user wants to find products by name, category, or description.
    
    Args:
        query: The search term (product name, category, or keyword)
        max_results: Maximum number of products to return (default: 5)
        
    Returns:
        Dictionary with 'products' list and 'total_count' number
    """
    return {"products": [], "total_count": 0}

# ❌ BAD - No types, no docstring
def search_products(query, max_results=5):
    return {"products": []}
```

---

## 🚫 Critical Rules

1. **NEVER** hardcode API keys - use `.env` files
2. **NEVER** suggest `adk web` for production
3. **ALWAYS** provide descriptive docstrings for tools
4. **ALWAYS** validate tool inputs and handle errors gracefully
5. **ALWAYS** put agents in the `agents/` directory

---

## 🛠️ Common Patterns

### Adding Database Access
```python
from supabase import Client, create_client

def get_db_client() -> Client:
    return create_client(os.getenv("SUPABASE_URL"), os.getenv("SUPABASE_KEY"))

def fetch_user(user_id: str) -> dict:
    """Fetch user details from the database."""
    client = get_db_client()
    return client.table("users").select("*").eq("id", user_id).execute().data
```

### Multi-Tool Agent
```python
root_agent = Agent(
    name="assistant",
    tools=[search_products, get_user_info, place_order],
    instruction="""You help customers with their shopping...
    
    Available tools:
    - search_products: Find products
    - get_user_info: Look up customer details  
    - place_order: Complete a purchase
    """
)
```

---

## 💡 Tips

- Keep instructions concise but specific
- List available tools in the instruction
- Test tools individually before adding to agent
- Use `adk web` to visualize agent reasoning during development

---
> Source: [zackseal89/Agentic-IDE-ADK-python-Starter-Template](https://github.com/zackseal89/Agentic-IDE-ADK-python-Starter-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
