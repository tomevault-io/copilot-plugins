## jarvisassistant

> Jarvis is a sophisticated, modular AI assistant built with Python. It leverages **Google's Gemini models** (via `pydantic-ai` and `google-genai`) for intelligence and **ChromaDB** for long-term vector memory. The system is designed with a **multi-agent architecture**, where a central router delegates tasks to specialized agents (Research, Media, Task Management, System Control, etc.).

# Jarvis AI Assistant

## Project Overview
Jarvis is a sophisticated, modular AI assistant built with Python. It leverages **Google's Gemini models** (via `pydantic-ai` and `google-genai`) for intelligence and **ChromaDB** for long-term vector memory. The system is designed with a **multi-agent architecture**, where a central router delegates tasks to specialized agents (Research, Media, Task Management, System Control, etc.).

## Architecture

### Core Components
- **Entry Point (`main.py`):** Initializes the system, sets up memory and logging, and runs the interactive REPL loop.
- **Router (`core/router.py`):** Uses a semantic classifier agent to analyze user requests and route them to the most appropriate specialist agent. It can also trigger a **Planning Agent** for complex multi-step tasks.
- **Base Agent (`core/base_agent.py`):** Provides the foundational structure for all specialized agents, handling configuration, model setup, and context injection.
- **Memory System (`memory/`):**
    - **Conversation Memory:** Short-term history stored in JSON.
    - **Vector Store:** Long-term semantic memory using ChromaDB and Gemini embeddings.
    - **User Profile:** Persistent user preferences and facts.
- **Model Gateway (`core/model_gateway.py`):** Manages model selection and settings for different agents.

### Specialized Agents
The system is composed of several specialized agents located in the `agents/` directory:
- **General Assistant:** Handles chit-chat and general queries.
- **Task Agent:** Manages to-do lists and task persistence.
- **Research Agent:** Performs web searches and information gathering using DuckDuckGo.
- **Media Agent:** Handles media playback and YouTube integration.
- **File Agent:** Manages file system operations within allowed directories.
- **System Agent:** Handles system-level controls (volume, brightness, etc.).
- **Planning Agent:** Breaks down complex goals into executable steps for other agents.
- **Scheduler Agent:** Manages reminders and scheduled tasks.

## Setup & Usage

### Prerequisites
- Python 3.10+
- Google Gemini API Key

### Installation
1. **Clone the repository.**
2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
3. **Environment Setup:**
   Create a `.env` file in the root directory with the following variables:
   ```env
   GEMINI_API_KEY=your_api_key_here
   GEMINI_MODEL=gemini-1.5-flash # or gemini-1.5-pro / gemini-2.0-flash-exp
   LOG_LEVEL=INFO
   ```

### Running the Assistant
Start the interactive REPL interface:
```bash
python main.py
```

### Usage Commands (REPL)
- `exit` or `quit`: Stop the assistant.
- `clear`: Clear the current conversation memory.

## Key Directories

| Directory | Description |
| :--- | :--- |
| `agents/` | Implementation of specialized agents (General, Media, Task, etc.). |
| `config/` | Configuration files (`settings.py`, `apps.yaml`) and agent prompts (`prompts/*.yaml`). |
| `core/` | Core framework logic: Routing, Base Agent, Registry, Logging, Context Building. |
| `data/` | Persistent data storage: ChromaDB, conversation history, user profiles, tasks. |
| `logs/` | System logs (`jarvis.log`). |
| `memory/` | Memory management logic (Vector Store, User Profile, Conversation Memory). |
| `services/` | External service integrations (Media Player, YouTube, System Control, Task Store). |

## Development Guidelines

- **Asynchronous First:** The entire codebase is built on `asyncio`. All I/O operations (file, network, DB) should be non-blocking.
- **Type Safety:** Rigorously use Python type hints and Pydantic models for data structures and function signatures.
- **Agent Framework:** Agents are built using `pydantic-ai`. New tools should be registered within the `register_tools` method of the specialist agent classes.
- **Prompt Management:** Agent system prompts and configurations are externalized in `config/prompts/` and `config/models.yaml`.
- **Logging:** Use the centralized logging configuration from `core.logging_config`. Avoid using `print()` for debugging in production-ready code.
- **Security:** The `FileAgent` enforces directory restrictions. Always check paths against `ALLOWED_ROOTS`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NehorayChalfon0166)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NehorayChalfon0166)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
