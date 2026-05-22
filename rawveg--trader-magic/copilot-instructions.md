## trader-magic

> - Run all tests: `python -m unittest discover -s tests`

# TraderMagic Development Guide

## Commands
- Run all tests: `python -m unittest discover -s tests`
- Run single test: `python -m unittest tests.test_ai_decision`
- Run specific test method: `python -m unittest tests.test_ai_decision.TestAIDecisionService.test_analyze_rsi_buy`
- Start services: `docker compose up -d`
- Restart all services: `./restart.sh`

## Code Style
- **Imports**: Standard library first, followed by third-party, then local modules
- **Type Annotations**: Use typing module; all function parameters and returns should be typed
- **Exception Handling**: Use specific exceptions and proper logging; avoid bare except blocks
- **Naming**: snake_case for variables/functions, PascalCase for classes, UPPER_CASE for constants
- **Docstrings**: Include for all public functions and classes; specify parameters and return types
- **Async Code**: Properly await async operations; use asyncio.run for top-level execution
- **Dependency Imports**: Import within functions when handling circular dependencies
- **Error Handling**: Log errors with appropriate log levels; provide context in error messages

## Architecture Patterns
- Service-based architecture with clear separation of concerns
- Redis for inter-service communication
- Config-driven with environment variables via dotenv

---
> Source: [rawveg/trader-magic](https://github.com/rawveg/trader-magic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
