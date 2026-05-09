## coding-conventions

> - **All code comments, docstrings, and error messages must be in English**

# Coding Conventions

## General conventions
- **All code comments, docstrings, and error messages must be in English**

## Language
- Use Python 3.12
- Use **uv** for package management
- Follow PEP 8 style guidelines

## Code Style
- Use **type hints** for all functions
- Use **async/await** pattern for I/O operations
- Use structured error responses following **OpenAI format**
- Log errors with appropriate levels

## Architecture Patterns
- **OpenAI Compatibility**: Strict adherence to OpenAI API format
- **Async Execution**: Non-blocking CLI execution with concurrency control
- **Fake Streaming**: Split complete responses line-by-line for streaming effect
- **Centralized Configuration**: All config in [src/gemini_cli_proxy/config.py](mdc:src/gemini_cli_proxy/config.py)

---
> Source: [nettee/gemini-cli-proxy](https://github.com/nettee/gemini-cli-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
