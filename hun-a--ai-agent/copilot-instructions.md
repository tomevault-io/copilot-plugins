## ai-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python-based AI agent project built with CrewAI framework. The project uses OpenAI's models (specifically o4-mini) and integrates with various LangChain components for building AI agents and workflows.

## Development Setup

### Package Manager
This project uses `uv` as the package manager (evident from uv.lock file).

### Environment Setup
- Copy environment variables from `.env` file which contains:
  - `OPENAI_API_KEY` - OpenAI API key for model access
  - `SERPER_API_KET` - Serper API key for search functionality (note: typo in variable name)
- Python 3.12+ required

### Dependencies
Key dependencies include:
- `crewai` - Main framework for multi-agent AI systems
- `langchain` ecosystem - For LLM integration and tools
- `openai` - OpenAI API client
- `python-dotenv` - Environment variable management
- `yfinance` - Financial data integration

## Development Commands

### Installation
```bash
uv sync
```

### Running the Application
```bash
python main.py
```

## Code Architecture

### Entry Point
- `main.py` - Simple entry point that loads environment variables and sets up OpenAI model configuration

### Configuration
- Model configuration is set via environment variable: `OPENAI_MODEL_NAME = "o4-mini"`
- Environment variables loaded via `python-dotenv`

## Key Framework Concepts

This project is built around CrewAI which provides:
- `Agent` - Individual AI agents with specific roles
- `Task` - Specific tasks agents can perform  
- `Crew` - Coordination of multiple agents working together

The incomplete main.py suggests this is a starter template for building multi-agent AI systems.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hun-a)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hun-a)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
