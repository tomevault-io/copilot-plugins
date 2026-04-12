## effective-context-length

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python CLI tool for testing and determining the effective context length limits of OpenAI-compatible LLM endpoints. The tool systematically tests various context lengths, collects error rate data, and provides comprehensive reporting on practical limits.

## Development Commands

### Running Tests
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_http_client.py

# Run with verbose output
pytest -v

# Run tests with coverage
pytest --cov=effective_context_length
```

### Installing Dependencies
```bash
pip install -r requirements.txt
```

### Running the Tool
```bash
# Basic usage
python effective_context_length.py https://api.openai.com/v1

# With specific model and max tokens
python effective_context_length.py http://localhost:8000/v1 --model custom-model --max-tokens 128000

# With binary search strategy
python effective_context_length.py https://api.example.com/v1 --strategy binary-search --output results.json

# With API key
python effective_context_length.py https://api.openai.com/v1 --api-key sk-... --model gpt-4 --verbose
```

## Architecture

### Core Components

1. **CLI Interface** (`effective_context_length.py`)
   - Entry point with argument parsing using argparse
   - Handles authentication, configuration, and logging setup

2. **Core Engine** (`effective_context_length/tester.py`)
   - Main testing logic with multiple strategies (hybrid, gradual, binary-search, predefined)
   - Manages test sessions and orchestrates test execution
   - Abstract base class for extensible testing strategies

3. **HTTP Client** (`effective_context_length/http_client.py`)
   - API communication using httpx with retry logic and rate limiting
   - Handles OpenAI-compatible API requests and responses
   - Error categorization and handling

4. **Context Generator** (`effective_context_length/context_generator.py`)
   - Generates test payloads of varying context lengths
   - Multiple content strategies (tokens, words, sentences, mixed)
   - Accurate token counting using tiktoken

5. **Results Analyzer** (`effective_context_length/analyzer.py`)
   - Statistical analysis of test results
   - Effective context length determination with confidence intervals
   - Error rate calculations and quality scoring

6. **Utilities** (`effective_context_length/utils.py`)
   - Progress reporting, timing, and formatting utilities
   - Configuration management and logging helpers

### Key Design Patterns

- **Strategy Pattern**: Testing strategies (hybrid, gradual, binary-search) are implemented as separate strategies
- **Async/Await**: All HTTP requests are asynchronous for better performance
- **Dataclasses**: Structured data representation for test results and analysis
- **Dependency Injection**: Components are loosely coupled with clear interfaces

### Testing Architecture

Tests are organized by component:
- `test_http_client.py`: HTTP client functionality and error handling
- `test_context_generator.py`: Context generation and token counting
- `test_integration.py`: End-to-end integration tests

## Important Implementation Details

1. **Token Counting**: Uses tiktoken library for accurate token counting, critical for determining context length limits
2. **Error Handling**: Comprehensive error categorization (HTTP, timeout, token, rate limit, content filter, etc.)
3. **Retry Logic**: Built-in retry mechanism with exponential backoff for network requests
4. **Rate Limiting**: Respects API rate limits and includes cooldown periods
5. **Statistical Analysis**: Uses scipy/numpy when available for confidence interval calculations
6. **Multiple Output Formats**: Supports JSON, CSV, and text-based reports

## Development Notes

- The tool is designed to be extensible - new testing strategies can be added by extending the abstract base class
- All HTTP communication is done through the HTTPClient abstraction for consistent behavior
- The architecture supports both OpenAI and custom OpenAI-compatible endpoints
- Error thresholds and confidence levels are configurable based on use case requirements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ffreemt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ffreemt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
