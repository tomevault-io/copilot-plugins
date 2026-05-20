## ai-mock-interviewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI Mock Interviewer application built with CrewAI that simulates realistic tech interviews using multi-agent conversations. The system creates technical interview questions, evaluates user responses, and provides follow-up questions with feedback.

## Architecture

The application follows a multi-agent CrewAI architecture with two main interfaces:

### Core Components

1. **Multi-Agent System**: Built on CrewAI framework with specialized agents:
   - `company_researcher`: Researches companies and interview practices
   - `question_preparer`: Creates technical questions with model answers
   - `answer_evaluator`: Evaluates user responses against correct answers
   - `follow_up_questioner`: Generates contextual follow-up questions

2. **Data Models**: Uses Pydantic for structured data:
   - `QuestionAnswerPair`: Schema for questions and their correct answers

3. **Two Main Crews**:
   - **Preparation Crew**: Company research + question generation (sequential process)
   - **Evaluation Crew**: Answer assessment + follow-up generation (can run async)

### Application Interfaces

1. **CLI Version** (`ai_mock_interviewer/interview_practice_system.py`):
   - Command-line interface for mock interviews
   - Async execution for optimal performance
   - Main entry point with hardcoded example (Google Data Scientist role)

2. **Streamlit Web App** (`ai_mock_interviewer/chatbot_ui.py`):
   - Interactive web interface with chat-style UI
   - Voice input support using Whisper for speech-to-text
   - Real-time question generation and evaluation
   - Session state management for interview flow

3. **Jupyter Notebook** (`Demo.ipynb`):
   - Interactive demonstration and experimentation environment
   - Step-by-step walkthrough of the agent system

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync

# Install with development dependencies
uv sync --group dev
```

### Running the Application
```bash
# CLI version
uv run ai_mock_interviewer/interview_practice_system.py

# Streamlit web app
uv run streamlit run ai_mock_interviewer/chatbot_ui.py

# Jupyter notebook for experimentation
uv run jupyter lab Demo.ipynb
```

### Code Quality and Linting
```bash
# Check for linting issues
uv run --group dev ruff check .

# Auto-fix linting issues
uv run --group dev ruff check --fix .

# Format code
uv run --group dev ruff format .
```

## Key Dependencies

- **CrewAI**: Multi-agent framework for orchestrating AI agents
- **Streamlit**: Web app framework for the UI
- **OpenAI Whisper**: Speech-to-text for voice input
- **Pydantic**: Data validation and serialization
- **SerperDevTool**: Web search capabilities for company research

## Configuration

- **Python Version**: 3.10-3.11 (specified in pyproject.toml)
- **Linting**: Ruff configuration in `ruff.toml` with line length 88
- **Package Management**: Uses uv for dependency management and execution

## File Structure

```
ai_mock_interviewer/
├── interview_practice_system.py  # Core CrewAI logic + CLI interface
├── chatbot_ui.py                 # Streamlit web interface
Demo.ipynb                        # Jupyter notebook demo
pyproject.toml                    # Project configuration
ruff.toml                         # Linting rules
```

## Important Notes

- The system requires external API keys for CrewAI agents and search functionality
- Voice input feature downloads Whisper models on first use
- Async operations are used for performance optimization in follow-up question generation
- Session state in Streamlit manages the interview flow and prevents duplicate question generation

---
> Source: [pigna90/ai-mock-interviewer](https://github.com/pigna90/ai-mock-interviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
