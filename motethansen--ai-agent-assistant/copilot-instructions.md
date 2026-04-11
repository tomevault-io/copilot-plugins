## ai-agent-assistant

> This project is an automated "AI Assistant" that synchronizes tasks from local Markdown files (Obsidian/Logseq) and Apple Reminders with a Google Calendar. It features a multi-agent architecture designed for privacy and performance, prioritizing local LLMs (Ollama/OpenClaw).

# GEMINI.md: AI-Powered Markdown-Calendar-AI Bridge

## Project Overview
This project is an automated "AI Assistant" that synchronizes tasks from local Markdown files (Obsidian/Logseq) and Apple Reminders with a Google Calendar. It features a multi-agent architecture designed for privacy and performance, prioritizing local LLMs (Ollama/OpenClaw).

### Main Technologies
- **Python 3.11+**: Core programming language.
- **Watchdog**: Real-time monitoring of Markdown file changes.
- **Google Calendar API**: Syncing schedules and events.
- **Chromadb**: Local vector database for RAG-based context retrieval.
- **Streamlit**: Web-based "Mission Control" dashboard.
- **LangChain & Ollama**: Integration for advanced agentic workflows and local LLMs.
- **Local LLMs**: Qwen 3.5 (9B), Ollama, and OpenClaw for private task processing.
- **Cloud LLMs**: Gemini, OpenAI, and Claude (Optional).

## Architecture
1. **Observer Layer (`observer.py`)**: Listens for modifications to Obsidian and LogSeq files. Specifically extracts LogSeq tasks marked with `LATER`.
2. **Multi-Agent Orchestrator (`ai_orchestration.py`)**: Routes requests between local and cloud models based on task complexity and server health.
3. **Core Specialized Agents**:
    - **Monitoring Agent (`monitoring_agent.py`)**: Tracks the pulse of local AI servers.
    - **Calendar Agent (`calendar_agent.py`)**: Periodically caches calendar data to `datainput/googlecalendar.yml`.
    - **Planning Agent (`planning_agent.py`)**: Finalizes schedules and updates local notes.
    - **RAG Agent (`rag_agent.py`)**: Retrieves deep context from your "Second Brain".
    - **Book Agent (`book_agent.py`)**: Indexes and searches PDF/EPUB libraries.
    - **Travel Agent (`travel_agent.py`)**: Researches trips via Google Search grounding.
    - **Gmail Agent (`gmail_agent.py`)**: Integrates email context into daily plans.

## Building and Running

### Prerequisites
- Python 3.11+ (Recommended for Google GenAI compatibility)
- Google Cloud Project with Calendar and Gmail APIs enabled.
- `credentials.json` (OAuth Client ID) in the root directory.
- **Docker/OrbStack**: Required to run the local OpenClaw server.

### Setup & Management
```bash
# Guided one-click installation
./install.command  # (Mac)
./install.sh       # (Linux)

# Automated configuration wizard
make setup
```

### Running the Project
```bash
# Start background observer and calendar sync
make run

# Start interactive chat mode
make run-chat

# Launch the Web Mission Control
make run-ui
```

## Development Conventions
- **Local-First**: The system defaults to Ollama/OpenClaw to protect user data.
- **Cached Context**: Google Calendar data is cached locally to speed up AI interactions.
- **Confirmed Actions**: Any file system change or booking requires human confirmation.
- **Timezone Aware**: All timestamps include local offsets to prevent scheduling shifts.

## Key Files
- `main.py`: Entry point and CLI orchestration.
- `app.py`: Streamlit Web UI and Mission Control.
- `install.sh`: Self-repairing installation and verification script.
- `scripts/manage_services.sh`: Local AI service manager (Ollama/OpenClaw).
- `scripts/check_ai_working.py`: Automated AI functionality verification.
- `update_manager.py`: Health monitoring and background update tracking.
- `ai_orchestration.py`: Priority-based LLM routing and RAG context retrieval.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/motethansen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/motethansen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
