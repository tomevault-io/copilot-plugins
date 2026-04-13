## ai-chat-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Foundation is a real-time chat application that mimics ChatGPT's interface. The main application is located in the `sse-chatgpt/` directory and consists of a FastAPI backend that integrates with Google Gemini LLM via LangChain, and a vanilla JavaScript frontend.

## Commands

### Setup and Run
```bash
cd sse-chatgpt
python3 -m venv .venv
source .venv/bin/activate  # On macOS/Linux
pip install -r requirements.txt

# Configure environment
# Ensure .env file contains: GOOGLE_API_KEY=your_gemini_api_key

# Run the application
uvicorn main:app --reload --app-dir .

# Run on different port
uvicorn main:app --reload --app-dir . --port 8001
```

### Testing
```bash
# Install development dependencies (includes pytest and testing tools)
pip install -r requirements-dev.txt

# Run all tests
pytest tests/ -v

# Run tests with coverage report
pytest tests/ --cov=main --cov-report=term-missing

# Run tests with HTML coverage report
pytest tests/ --cov=main --cov-report=html
# View coverage report: open htmlcov/index.html

# Run specific test file
pytest tests/unit/test_session_management.py -v

# Run tests matching a pattern
pytest tests/ -k "session" -v

# Run tests in parallel (faster)
pytest tests/ -n auto

# Generate JUnit XML report for CI
pytest tests/ --junitxml=test-results.xml
```

### Code Quality and Linting

The project uses multiple linting tools to maintain code quality (same as GitHub Actions CI):

```bash
# Quick formatting (recommended at end of work sessions)
./format.sh

# Comprehensive linting with all tools
python3 lint.py

# Check-only mode (CI-compatible)
python3 lint.py --check

# Individual tools:
black .                    # Code formatting
isort .                    # Import sorting  
mypy main.py --ignore-missing-imports    # Type checking
bandit -r . -f json -o bandit-report.json  # Security linting
```

**Important**: Always run `./format.sh` or `python3 lint.py` after completing development work to ensure code passes CI formatting checks.

## Architecture

### High-Level Flow
```
Browser (JavaScript) ↔ FastAPI Backend ↔ Google Gemini API
         ↓                    ↓                 ↓
   - UI interactions    - Session mgmt    - LLM responses
   - File uploads       - SSE streaming   - Title generation
   - Real-time display  - Message history
```

### Key Components

1. **Backend (`main.py`)**:
   - FastAPI server with in-memory session storage
   - Integrates with Google Gemini via LangChain
   - Handles multimodal inputs (text, images, PDFs) using base64 encoding
   - Streams responses using Server-Sent Events (SSE)

2. **Frontend (`static/script.js`, `templates/index.html`)**:
   - Single-page application with session management
   - Handles real-time streaming via EventSource API
   - File upload and preview functionality
   - Markdown rendering with syntax highlighting

### API Endpoints
- `GET /` - Main chat interface
- `POST /stream` - Stream chat responses (accepts session_id, prompt, file)
- `GET /sessions` - List all sessions
- `GET /sessions/{session_id}` - Get chat history
- `DELETE /sessions/{session_id}` - Delete session

### Important Implementation Details

1. **File Handling**: All files (images, PDFs, text) are sent to Gemini using the `image_url` format with base64 encoding and appropriate MIME types.

2. **Session Management**: Sessions are stored in-memory in the `chat_sessions` dictionary. Each session has a UUID, title, and message history.

3. **Streaming Pattern**: The `/stream` endpoint returns Server-Sent Events with:
   - `data: <content>` for streaming tokens
   - `event: title\ndata: <title>` for title updates
   - `data: [DONE]` to signal completion

4. **Hardcoded Paths**: Template and static directories use absolute paths that may need adjustment for different environments.

## Coding Guidelines

From GEMINI.md:
- Follow existing JavaScript coding style for frontend code
- All JavaScript code goes in the static folder
- Follow strict Python guidelines for backend code
- Prefer functional programming paradigms where appropriate
- Avoid introducing new external dependencies unless absolutely necessary

## Current Limitations

- No persistent storage (sessions lost on restart)
- No user authentication
- Hardcoded absolute paths in main.py
- No automated tests or linting configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eduardoarantes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eduardoarantes)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
