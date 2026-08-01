## thenzt-open-source

> The `src/ai/agents/` folder contains the **core agent implementations** for the Insight Bot multi-agent system.

# Agents

## Overview

The `src/ai/agents/` folder contains the **core agent implementations** for the Insight Bot multi-agent system.
Each agent is a specialized, modular Python class, responsible for a specific type of reasoning or data operation (e.g., finance, code, search, planning, etc.).
All agents inherit from a shared `BaseAgent`, ensuring a consistent interface and extensibility.

---

## Agent Architecture

* **BaseAgent:**
  All agents derive from `BaseAgent`, enforcing core methods for prompt formatting, system prompt handling, and invocation logic.
* **Modularity:**
  Each agent addresses a distinct function or query type, making the system flexible, maintainable, and easy to extend.
* **Prompt-driven:**
  Each agent uses its own prompt (imported from `src/ai/agent_prompts/`) and may access specific tools, models, or schemas.

---

### Key Agents and Their Responsibilities

| Agent Name                  | Description                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------- |
| **BaseAgent**               | Abstract class; all agents inherit from this, providing shared structure and method requirements.  |
| **IntentDetector**          | Classifies the user's query intent (e.g., finance, code, search) to kickstart the workflow.        |
| **PlannerAgent**            | Breaks down complex queries into subtasks, generating a structured research/task plan.             |
| **ManagerAgent**            | Analyzes context, orchestrates the sequence of agent tasks, and manages the workflow dynamically.  |
| **ExecutorAgent**           | Executes the task plan, generates, and modifies task lists as needed.                              |
| **DBSearchAgent**           | Handles semantic search over the internal database (Qdrant, etc.).                                 |
| **WebSearchAgent**          | Conducts internet research via web search tools and APIs.                                          |
| **FinanceDataAgent**        | Fetches, analyzes, and processes finance-specific data from APIs, databases, or scrapers.          |
| **DataComparisonAgent**     | Compares datasets or results from multiple sources for consistency and insight.                    |
| **SentimentAnalysisAgent**  | Analyzes sentiment from news, reports, or social media content.                                    |
| **SocialMediaAgent**        | Gathers and analyzes social media data and trends relevant to the query.                           |
| **ResponseGenerationAgent** | Synthesizes and formats final answers, incorporating graphs, charts, and structured output.        |
| **TaskValidator**           | Validates the correctness and quality of responses for each task.                                  |
| **ValidationAgent**         | Validates the final proposed response before it is shown to the user, can trigger correction loop. |
| **Summarizer**              | Summarizes or elaborates responses as needed (usually streaming/async, can add examples).          |
| **FastAgent**               | Optimized agent for fast, simple queries using minimal steps and direct tool access.               |
| **Utils**                   | Shared helper functions for context passing, routing, formatting, and other agent logic.           |

---

### Agent Implementation Pattern

Each agent generally follows this structure:

* **Imports tools, prompts, config, and schemas needed for its operation.**
* **Implements `format_input_prompt(state)`** to generate the prompt for the LLM based on the current task.
* **Implements `__call__(state)`** which:

  * Invokes the language model with system and human messages.
  * Calls external tools or APIs as needed.
  * Handles context, feedback, retries, and fallback models.
  * Returns a standardized `Command` or state update for orchestration.

---

#### Example: Typical Agent Lifecycle

1. **Receives state and current task from the graph.**
2. **Formats input prompt using task, history, and context.**
3. **Invokes LLM (and/or tools) for reasoning, processing, or generation.**
4. **Processes results, adds to task messages, and returns a Command** to either move to the next agent or finish.

---

#### Sample Pseudocode

```python
class ExampleAgent(BaseAgent):
    def __init__(self):
        super().__init__()
        self.model = get_llm(config.MODEL, config.TEMPERATURE)
        self.system_prompt = SYSTEM_PROMPT

    def format_input_prompt(self, state):
        # Build task-specific prompt string
        ...

    def __call__(self, state):
        # Build messages
        input_prompt = self.format_input_prompt(state)
        system_message = SystemMessage(content=self.system_prompt)
        human_message = HumanMessage(content=input_prompt)
        # LLM call and response handling
        ...
        return Command(goto=next_agent, update={...})
```

---

### Coordination and Routing

* **Task Routing**:
  `src/backend/utils/utils.py` contains logic (`task_router_node`, etc.) for sending tasks to the appropriate agent based on the workflow.
* **Context Handling**:
  Helper functions (e.g., `get_context_messages`) ensure agents have access to relevant prior messages or context.
* **Validation and Correction**:
  Some agents (e.g., `ValidationAgent`, `TaskValidator`) can trigger correction cycles or manual intervention if outputs are incomplete or incorrect.

---

### Error Handling and Model Fallback

* Agents can **fall back to alternate models** if the primary LLM fails or returns an error.
* Most agents implement try/except logic to ensure robust, fail-safe operations.

---

### Extending Agents

To add a new agent:

1. Create a new agent file/class inheriting from `BaseAgent`.
2. Implement the required methods and import any needed tools or prompts.
3. Register the agent in the agent graph (`insight_graph.py`).
4. Add corresponding prompt and schema files as needed.

---

### File Reference

* Each agent class is in its own file in `src/ai/agents/`.
* Shared logic and base class: `base_agent.py`
* Routing/utilities: `src/backend/utils/utils.py`
* Example agents: `finance_data_agent.py`, `web_search_agent.py`, `manager_agent.py`, etc.

---
> Source: [IAI-solution/TheNZT_Open_Source](https://github.com/IAI-solution/TheNZT_Open_Source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
