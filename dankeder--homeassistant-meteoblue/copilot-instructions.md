## homeassistant-meteoblue

> HomeAssistant-MeteoBlue is a Home Assistant integration for MeteoBlue. It reads

# CLAUDE.md

## Project overview

HomeAssistant-MeteoBlue is a Home Assistant integration for MeteoBlue. It reads
weather foreccast data from the MeteoBlue Forecast API and Image API and
exposes it to Home Assistant.

- **Language:** Python 3.14+
- **Key dependencies:** httpx, home-assistant
- **Build tool:** uv

## General instructions

- Be concise in responses
- Make the changes you make logically coherent. Do not make "big bang" changes.
If the user asks to implement a feature, big refactoring or other complicated
change divide the change into multiple smaller ones and ask the user whether you
can implement them one step at a time, giving him an opportunity to review the
changes incrementally.

## Build/Lint/Test Commands

### Run

```bash
scripts/run
```

### Lint and format

```bash
scripts/lint
```

### Test

```bash
scripts/test
```

## Code Style Guidelines

### Imports

- Use standard library imports first, then third-party imports, then local imports
- Sort imports alphabetically within each category
- Use relative imports for local modules

### Formatting

- Follow PEP 8 guidelines
- Use 4 spaces for indentation
- Maximum line length of 88 characters
- Use ruff for code formatting

### Types

- Use type hints for all function parameters and return values
- Prefer `typing` module for complex types
- Use `Optional` for optional parameters

### Naming Conventions

- Use `snake_case` for functions and variables
- Use `PascalCase` for classes
- Use `UPPER_CASE` for constants
- Avoid single character variable names except for loop counters

### Error Handling

- Use specific exceptions instead of generic `Exception`
- Log errors with meaningful context
- Use try/except blocks around potentially failing operations
- Validate inputs early and fail fast

### Documentation

- Use docstrings for all public functions and classes
- Follow Google Python Style Guide for docstrings
- Include type hints in docstrings when needed

### Project Structure

- All source code in `custom_components/` directory
- Home Asisstant config stub for development is in `config/`
- Use `__init__.py` for package initialization

### MeteoBlue API documentation

- Forecast API:
    - Docs: https://docs.meteoblue.com/en/weather-apis/forecast-api/overview
    - OpenAPI specification: https://my.meteoblue.com/packages/redoc#tag/Overview-Structure
- Account API:
    - Docs: https://docs.meteoblue.com/en/weather-apis/further-apis/account-api

---
> Source: [dankeder/HomeAssistant-MeteoBlue](https://github.com/dankeder/HomeAssistant-MeteoBlue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
