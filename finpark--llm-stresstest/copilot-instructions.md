## llm-stresstest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LLM Stress Test Tool - A robust Python application for testing Large Language Models (LLMs) performance and hardware requirements. The tool sends configurable batches of questions to LLM APIs and collects detailed performance metrics including automatic quality evaluation of responses.

## Development Commands

```bash
# Activate virtual environment
source .venv/bin/activate

# Install/update dependencies
uv sync

# Run the stress test (filename auto-generated from config)
python llm_stresstest.py

# Add new dependencies
uv add <package_name>
```

## Project Structure

```
llm_stresstest/
├── config/
│   ├── config.json     # Test configuration (questions count, concurrency, timeout, etc.)
│   └── questions.json  # Question bank (234 questions in German)
├── logs/               # Log files with timestamps
├── results/            # Output directory for test results
├── llm_stresstest.py   # Main application with async processing
├── llm_auswertung.py   # Streamlit dashboard for analysis
└── projectplan.md      # Detailed project planning and architecture
```

## Architecture

- **Async Processing**: Uses `asyncio` and `aiohttp` for concurrent request handling
- **OpenAI-Compatible API**: Works with any OpenAI-compatible endpoint (Ollama, vLLM, etc.)
- **Quality Evaluation**: Integrated automatic quality assessment with 8 metrics
- **Robust Error Handling**: Comprehensive try-catch blocks, no crashes on errors
- **Detailed Logging**: File and console logging with timestamps
- **Structured Output**: JSON results with metadata, individual results, quality metrics, and aggregates

## Configuration (config/config.json)

```json
{
    "questions": 5,           // Number of questions to process
    "concurrent": 1,          // Parallel requests (1 = sequential)
    "url": "http://...",     // LLM API endpoint
    "server_name": "Server 1", // Descriptive server name for reports
    "model": "model_name",   // Model identifier
    "timeout": 120.0,        // Request timeout in seconds
    "max_keepalive_connections": 20  // Connection pool size
}
```

## Output Format

Results are saved in `results/<filename>.json` with:
- **meta**: Test metadata (timestamps, server, model, duration)
- **results**: Individual question results (question, answer, time, tokens, quality, quality_metrics)
- **aggregate**: Statistics (sum, avg, min, max for runtime, tokens, and quality)

## Error Handling

- Connection failures abort immediately with clear error messages
- Individual question failures don't stop the test
- Emergency result saving if primary save fails
- All errors logged with stack traces in debug mode

## Testing Considerations

- Always verify connection before running tests
- Monitor log files for detailed execution info
- Check results JSON for error messages in answers
- Use concurrent > 1 for stress testing server capacity
- Quality evaluation is automatic; check quality scores in results
- Optional NLP dependencies: spacy, sentence-transformers for enhanced analysis

## Quality Evaluation Features

The integrated Quality Evaluator assesses each LLM response using:

### 8 Quality Metrics (0-10 scale):
- **Structure**: Logical organization and flow
- **Readability**: Clarity and understandability
- **Completeness**: How thoroughly the question is answered
- **Relevance**: Relevance to the asked question
- **Factual Consistency**: Accuracy of information
- **Language Flow**: Natural and fluent language
- **Coherence**: Internal logic and consistency
- **Overall Quality**: Comprehensive quality assessment

### Additional Metrics:
- Word count
- Sentence count  
- Average sentence length
- Repetition rate (most common words)

---
> Source: [FinPark/llm_stresstest](https://github.com/FinPark/llm_stresstest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
