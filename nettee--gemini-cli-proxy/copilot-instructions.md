## project-overview

> This is an OpenAI-compatible HTTP API wrapper for Google Gemini CLI tool, enabling seamless integration with OpenAI clients like Cherry Studio.

# Gemini CLI Proxy Project Overview

This is an OpenAI-compatible HTTP API wrapper for Google Gemini CLI tool, enabling seamless integration with OpenAI clients like Cherry Studio.

## Project Structure

The main entry point is [src/gemini_cli_proxy/cli.py](mdc:src/gemini_cli_proxy/cli.py), which handles command-line argument parsing and starts the FastAPI server defined in [src/gemini_cli_proxy/server.py](mdc:src/gemini_cli_proxy/server.py).

### Core Modules

- **CLI Entry**: [src/gemini_cli_proxy/cli.py](mdc:src/gemini_cli_proxy/cli.py) - Command-line interface using Click
- **FastAPI Server**: [src/gemini_cli_proxy/server.py](mdc:src/gemini_cli_proxy/server.py) - HTTP service with OpenAI-compatible endpoints
- **Gemini Client**: [src/gemini_cli_proxy/gemini_client.py](mdc:src/gemini_cli_proxy/gemini_client.py) - Async wrapper for Gemini CLI tool
- **OpenAI Adapter**: [src/gemini_cli_proxy/openai_adapter.py](mdc:src/gemini_cli_proxy/openai_adapter.py) - Format conversion between OpenAI and Gemini
- **Data Models**: [src/gemini_cli_proxy/models.py](mdc:src/gemini_cli_proxy/models.py) - Pydantic models for request/response validation
- **Configuration**: [src/gemini_cli_proxy/config.py](mdc:src/gemini_cli_proxy/config.py) - Application configuration management

### Key Files

- **Package Info**: [src/gemini_cli_proxy/__init__.py](mdc:src/gemini_cli_proxy/__init__.py) - Version and package metadata
- **Project Config**: [pyproject.toml](mdc:pyproject.toml) - Dependencies, build config, and CLI entry point
- **Documentation**: [README.md](mdc:README.md) - English documentation, [README_zh.md](mdc:README_zh.md) - Chinese documentation

## Key Features

- **OpenAI API Compatibility**: Implements `/v1/chat/completions`, `/v1/models`, and `/health` endpoints
- **Fake Streaming**: Splits complete Gemini CLI output line-by-line for OpenAI streaming compatibility
- **Rate Limiting**: 60 requests/minute using SlowAPI
- **Async Execution**: Non-blocking CLI execution with concurrency control
- **Modern Distribution**: Installable via `uvx` for zero-config execution

---
> Source: [nettee/gemini-cli-proxy](https://github.com/nettee/gemini-cli-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
