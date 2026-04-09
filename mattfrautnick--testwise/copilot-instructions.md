## testwise

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Testwise

Testwise is an LLM-powered test selection tool for CI/CD pipelines. It analyzes git diffs, uses an LLM (via litellm) to classify tests as `must_run`, `should_run`, or `skip`, then executes only the relevant tests. It ships as both a CLI (`testwise`) and a GitHub Actions composite action.

## Development Commands

```bash
# Install in dev mode
pip install -e ".[dev]"

# Run all tests
pytest

# Run tests with coverage
pytest --cov=testwise --cov-report=term-missing

# Run a single test file
pytest tests/test_parsers/test_pytest_parser.py

# Run a single test
pytest tests/test_parsers/test_pytest_parser.py::test_function_name

# Linting
ruff check src/ tests/
ruff format src/ tests/

# Type checking
mypy src/testwise/
```

## Architecture

The pipeline flows through these stages in order, orchestrated by `cli.py`:

1. **Config** (`config.py`) — Loads `.testwise.yml`, resolves defaults via `TestwiseConfig` Pydantic model
2. **Diff** (`diff_analyzer.py`) — Extracts git diff between base/head refs, filters and truncates
3. **Discovery** (`test_discovery.py`) — Finds test files matching runner glob patterns
4. **Parsing** (`test_discovery.py` → `parsers/`) — Delegates to parser plugins to extract individual test functions with metadata
5. **Context** (`context_builder.py`) — Assembles diff + parsed tests into LLM messages, respecting token budget
6. **LLM Selection** (`llm_selector.py`) — Calls LLM via litellm with structured output (falls back to text mode JSON extraction). Returns `LLMSelectionResponse` with per-test classifications
7. **Execution** (`test_runner.py`) — Runs selected tests using parser's `build_run_command()`
8. **Reporting** (`reporter.py`) — Outputs results as text, JSON, or GitHub Actions annotations

### Parser Plugin System

Parsers are registered via Python entry points (`testwise.parsers` group) and loaded at runtime by `parsers/__init__.py`. Each parser extends `BaseParser` ABC and implements:
- `parse_test_file()` — extracts `ParsedTest` objects with names, tags, `@covers` annotations, imports
- `build_run_command()` — builds the CLI command to run specific tests

Built-in parsers: `pytest` (test-level granularity) and `generic` (file-level fallback).

### Key Models (`models.py`)

All data flows through Pydantic models: `DiffResult` → `ParsedTestFile`/`ParsedTest` → `LLMSelectionResponse`/`TestSelection` → `TestResult` → `RunReport`. The `TestClassification` enum (`must_run`, `should_run`, `skip`) is central to the selection logic.

### LLM Fallback Strategy

`llm_selector.py` has a three-tier approach: (1) structured JSON output via `response_format`, (2) text mode with JSON schema in prompt, (3) raise `LLMError`. The caller in `cli.py` catches `LLMError` and falls back to running all tests when `fallback_on_error: true`.

## Configuration

- Config file: `.testwise.yml` (see `.testwise.example.yml` for all options)
- Uses litellm model format (e.g., `anthropic/claude-sonnet-4-20250514`, `openai/gpt-4o`)
- API key is read from the env var specified by `llm.api_key_env`

## Project Setup

- Python >=3.10, build system: hatchling
- Ruff config: line-length 100, target py310, rules: E, F, I, N, W, UP
- mypy: strict mode
- Tests directory: `tests/` with `fixtures/` and `test_parsers/` subdirectories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattfrautnick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mattfrautnick)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
